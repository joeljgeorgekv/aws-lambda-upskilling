# Performance, Pricing & Configuration

## Cold starts

A cold start happens when Lambda needs to create a new execution environment from scratch.

### What happens during a cold start

1. AWS downloads your deployment package
2. Creates a new execution environment (micro VM)
3. Initializes the runtime (Node.js, Python, Java, etc.)
4. Runs your initialization code (outside the handler)
5. Executes the handler

Steps 1 through 4 are the cold start overhead. Step 5 happens on every invocation.

### When cold starts happen

- First invocation of a new or updated function
- When Lambda scales up to handle more concurrent requests
- After an idle period (environments are recycled after ~5-15 minutes of inactivity)

### What affects cold start duration

| Factor | Impact |
| --- | --- |
| Runtime language | Java and .NET have longer cold starts than Node.js and Python |
| Deployment package size | Larger packages take longer to download and extract |
| Initialization code | Heavy setup logic (DB connections, SDK clients) adds time |
| Memory allocation | More memory = more CPU = faster initialization |
| VPC configuration | Functions inside a VPC used to have much longer cold starts (mostly resolved with Hyperplane ENI) |

### Typical cold start times

| Runtime | Typical cold start |
| --- | --- |
| Python | 100 - 300 ms |
| Node.js | 100 - 300 ms |
| Go | 50 - 200 ms |
| Java | 500 ms - 5 s |
| .NET | 400 ms - 2 s |

Java and .NET suffer the most due to JVM and CLR startup.

---

## SnapStart

SnapStart is a feature that dramatically reduces cold start times for **Java** Lambda functions (also available for .NET in preview).

### How SnapStart works

1. When you publish a new version of your function, Lambda **initializes** it
2. Lambda takes a **Firecracker microVM snapshot** of the initialized execution environment (memory + disk state)
3. On invocation, instead of initializing from scratch, Lambda **restores from the snapshot**

This skips the JVM startup and class loading phase entirely.

### Cold start comparison

| Scenario | Typical cold start |
| --- | --- |
| Java without SnapStart | 2 - 5 seconds |
| Java with SnapStart | 200 - 500 ms |

That is a **~10x improvement**.

### Enabling SnapStart

In the AWS Console:

1. Open your Lambda function
2. Go to **Configuration** → **General configuration**
3. Set **SnapStart** to `PublishedVersions`
4. Click **Save**
5. **Publish a new version** (SnapStart only works with published versions, not `$LATEST`)

Or in AWS SAM / CloudFormation:

```yaml
Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: com.example.App::handleRequest
      Runtime: java21
      SnapStart:
        ApplyOn: PublishedVersions
```

### Example: Java Lambda with and without SnapStart

This example simulates a function that loads heavy dependencies during init.

```java
package com.example;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;

import java.util.Map;
import java.util.HashMap;

public class App implements RequestHandler<Map<String, Object>, Map<String, Object>> {

    // This initialization runs during cold start
    // With SnapStart, it runs once at publish time and is restored from snapshot
    private static final long INIT_START = System.currentTimeMillis();
    private final Map<String, String> configCache;

    public App() {
        // Simulate heavy initialization: loading config, SDK clients, etc.
        this.configCache = new HashMap<>();
        for (int i = 0; i < 10000; i++) {
            configCache.put("key-" + i, "value-" + i);
        }
        // Simulate class loading and dependency injection overhead
        try {
            Thread.sleep(500); // represents real init work (DI, DB pool, etc.)
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        long initDuration = System.currentTimeMillis() - INIT_START;
        System.out.println("Init completed in " + initDuration + " ms");
    }

    @Override
    public Map<String, Object> handleRequest(Map<String, Object> event, Context context) {
        long handlerStart = System.currentTimeMillis();

        String name = (String) event.getOrDefault("name", "World");
        String greeting = "Hello, " + name + "! Cache size: " + configCache.size();

        long handlerDuration = System.currentTimeMillis() - handlerStart;

        Map<String, Object> response = new HashMap<>();
        response.put("statusCode", 200);
        response.put("greeting", greeting);
        response.put("handlerDurationMs", handlerDuration);
        response.put("initDurationMs", System.currentTimeMillis() - INIT_START);

        System.out.println("Handler completed in " + handlerDuration + " ms");
        return response;
    }
}
```

**Without SnapStart** — CloudWatch will show:

```text
REPORT Duration: 15 ms  Billed Duration: 100 ms  Init Duration: 2800 ms
```

The `Init Duration` of ~2800 ms is the cold start penalty.

**With SnapStart** — CloudWatch will show:

```text
REPORT Duration: 12 ms  Billed Duration: 100 ms  Restore Duration: 180 ms
```

`Restore Duration` replaces `Init Duration`, and it is typically **200-500 ms** instead of seconds.

### SnapStart caveats

- Only works with **published versions**, not `$LATEST`
- Uniqueness: random values, UUIDs, or connections established during init will be **shared across restored instances**. Use the `beforeCheckpoint` and `afterRestore` hooks (CRaC API) to handle this
- Does not support provisioned concurrency (you use one or the other)
- Currently available for **Java 11, Java 17, and Java 21** runtimes

### CRaC hooks for uniqueness

```java
import com.amazonaws.services.lambda.runtime.RequestHandler;
import org.crac.Context;
import org.crac.Core;
import org.crac.Resource;

import java.util.Map;

public class App implements RequestHandler<Map<String, Object>, Map<String, Object>>, Resource {

    private String uniqueId;

    public App() {
        Core.getGlobalContext().register(this);
        this.uniqueId = java.util.UUID.randomUUID().toString();
    }

    @Override
    public void beforeCheckpoint(Context<? extends Resource> context) {
        // Called before snapshot — clean up connections, clear sensitive state
    }

    @Override
    public void afterRestore(Context<? extends Resource> context) {
        // Called after restore — regenerate unique values, reconnect
        this.uniqueId = java.util.UUID.randomUUID().toString();
    }
}
```

---

## JVM Frameworks for Lambda: Feral & Quarkus

Since we run **Scala** in our systems, these two frameworks are directly relevant to how we build and optimize Lambda functions on the JVM.

### Feral (Typelevel)

[Feral](https://github.com/typelevel/feral) is a **Scala-first, purely functional** serverless framework built on the [Typelevel](https://typelevel.org/) ecosystem (Cats Effect, fs2, http4s).

#### Why Feral matters for us

- Written **in Scala, for Scala** — not a Java framework with Scala interop bolted on
- Built on **Cats Effect 3** — if your codebase already uses Cats Effect, http4s, or fs2, Feral slots in naturally
- Supports **AWS Lambda** natively with type-safe event/response models
- Provides `IOLambda` — a base class that wires up the Cats Effect `IO` runtime as the Lambda handler
- Supports **Scala.js** compilation to JavaScript, meaning your Scala Lambda can run on the **Node.js runtime** — this eliminates the JVM cold start entirely

#### How Feral works

Instead of implementing Java's `RequestHandler`, you extend `IOLambda` and write your handler in pure `IO`:

```scala
// build.sbt
libraryDependencies ++= Seq(
  "org.typelevel" %% "feral-lambda" % "0.3.1",
  "org.typelevel" %% "feral-lambda-api-gateway-proxy-http4s" % "0.3.1"
)
```

```scala
package com.example

import cats.effect.IO
import feral.lambda._
import feral.lambda.events._
import feral.lambda.apigatewayproxyv2._

object MyHandler extends IOLambda.Simple[
  ApiGatewayProxyEventV2,
  ApiGatewayProxyStructuredResultV2
] {

  override def apply(event: ApiGatewayProxyEventV2, context: Context[IO]): IO[Option[ApiGatewayProxyStructuredResultV2]] =
    IO.pure(Some(
      ApiGatewayProxyStructuredResultV2(
        statusCode = 200,
        headers = Map("Content-Type" -> "application/json"),
        body = Some(s"""{"message": "Hello from Feral! Path: ${event.rawPath}"}""")
      )
    ))
}
```

#### Feral with Scala.js (Node.js runtime — no JVM cold start)

This is Feral's killer feature for cold starts. You compile your Scala code to JavaScript and deploy it on the **Node.js Lambda runtime**:

```scala
// build.sbt — use Scala.js plugin
enablePlugins(ScalaJSPlugin)

// Use the scalajs variant of the feral dependencies
libraryDependencies ++= Seq(
  "org.typelevel" %%% "feral-lambda" % "0.3.1",
  "org.typelevel" %%% "feral-lambda-api-gateway-proxy-http4s" % "0.3.1"
)

scalaJSLinkerConfig ~= { _.withModuleKind(ModuleKind.CommonJSModule) }
```

Deploy the output `.js` file with the **Node.js 20.x runtime** instead of Java. Your handler code stays the same — pure Scala, pure Cats Effect — but cold starts drop from seconds to **~100-300 ms**.

#### Feral supported event types

| Event source | Feral type |
| --- | --- |
| API Gateway v2 (HTTP API) | `ApiGatewayProxyEventV2` |
| API Gateway v1 (REST API) | `ApiGatewayProxyRequestEvent` |
| SQS | `SqsEvent` |
| SNS | `SnsEvent` |
| S3 | `S3Event` |
| DynamoDB Streams | `DynamoDbStreamEvent` |
| CloudFormation Custom Resource | `CloudFormationCustomResourceRequest` |

---

### Quarkus

[Quarkus](https://quarkus.io/) is a **Kubernetes-native Java/Kotlin/Scala framework** designed for fast startup and low memory footprint. It has first-class AWS Lambda support and can compile to **GraalVM native images** for near-instant cold starts.

#### Why Quarkus matters for us

- Supports **Scala** via the JVM (Quarkus extensions run on any JVM language)
- Can compile to **GraalVM native images** — startup in **~10-50 ms** instead of seconds
- Has a dedicated `quarkus-amazon-lambda` extension with built-in packaging, SAM templates, and deployment scripts
- Works with **SnapStart** as well — you can combine Quarkus + SnapStart on Java if native image is not feasible
- Offers **dependency injection**, REST clients, database clients, and other enterprise features with minimal overhead

#### Quarkus Lambda with Scala

```scala
// build.sbt (or pom.xml equivalent)
// Quarkus is typically used with Maven/Gradle, but can work with sbt via the JVM classpath
```

```xml
<!-- pom.xml — add the Quarkus Lambda extension -->
<dependency>
  <groupId>io.quarkus</groupId>
  <artifactId>quarkus-amazon-lambda</artifactId>
</dependency>
```

```scala
package com.example

import com.amazonaws.services.lambda.runtime.{Context, RequestHandler}

case class InputEvent(name: String, action: String)
case class OutputResponse(statusCode: Int, message: String)

class ScalaLambdaHandler extends RequestHandler[InputEvent, OutputResponse] {

  // Quarkus handles DI and initialization at build time, not runtime
  // This dramatically reduces cold start even in JVM mode

  override def handleRequest(input: InputEvent, context: Context): OutputResponse = {
    val logger = context.getLogger
    logger.log(s"Processing: ${input.name} - ${input.action}")

    OutputResponse(
      statusCode = 200,
      message = s"Hello ${input.name}, action '${input.action}' processed"
    )
  }
}
```

#### Quarkus native image build

The biggest cold start advantage comes from GraalVM native compilation:

```bash
# Build native image for Lambda (uses Amazon Linux 2 compatible build)
./mvnw package -Pnative -Dquarkus.native.container-build=true

# This produces a bootstrap binary in target/
# Deploy as a "provided.al2023" custom runtime
```

With native images, the cold start drops dramatically:

| Mode | Typical cold start | Memory usage |
| --- | --- | --- |
| Quarkus on JVM | 1 - 3 seconds | ~150-250 MB |
| Quarkus on JVM + SnapStart | 200 - 500 ms | ~150-250 MB |
| Quarkus native image | 10 - 50 ms | ~30-80 MB |

#### Quarkus SAM template for native deployment

```yaml
Resources:
  MyScalaFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: not.used.in.native  # handler is baked into the binary
      Runtime: provided.al2023
      CodeUri: target/function.zip
      MemorySize: 256
      Timeout: 30
      Architectures:
        - x86_64
```

---

### Feral vs Quarkus: comparison for Scala teams

| Aspect | Feral | Quarkus |
| --- | --- | --- |
| **Language** | Scala-first (Typelevel) | Java-first (Scala via JVM) |
| **Programming model** | Pure FP — Cats Effect `IO` | Imperative / CDI-based |
| **Best cold start strategy** | Scala.js → Node.js runtime (~100-300 ms) | GraalVM native image (~10-50 ms) |
| **SnapStart compatible** | Not needed (use Scala.js instead) | Yes (JVM mode) |
| **Ecosystem fit** | Cats Effect, http4s, fs2, circe | Hibernate, RESTEasy, SmallRye, Vert.x |
| **Build tool** | sbt | Maven / Gradle |
| **Type safety** | Strong — typed event models in Scala | Standard Java generics |
| **Learning curve** | Low if you already use Typelevel | Low if you already use Java enterprise patterns |
| **Scala.js support** | ✅ First-class | ❌ Not supported |
| **Native image support** | ❌ Not the primary path | ✅ First-class |
| **Best for our team** | Functional Scala services, Typelevel stack | Services needing enterprise Java integrations |

### Which should we use?

```text
Already using Cats Effect / http4s / fs2?
  → Feral + Scala.js (Node.js runtime) — best cold starts, stays in the Typelevel world

Need enterprise integrations (DB pools, REST clients, DI)?
  → Quarkus + GraalVM native image — fastest absolute cold start (~10-50 ms)

Want the simplest migration path for existing Scala Lambda code?
  → Quarkus JVM mode + SnapStart — minimal code changes, good cold start improvement
```

---

## Pricing

Lambda pricing is based on **three dimensions**:

### 1. Number of requests

| Metric | Price |
| --- | --- |
| First 1 million requests per month | Free |
| Every additional 1 million requests | $0.20 |

### 2. Duration (compute time)

Duration is measured from when your handler starts executing until it returns or times out.

It is billed in **1 ms increments**.

| Memory | Price per ms |
| --- | --- |
| 128 MB | $0.0000000021 |
| 512 MB | $0.0000000083 |
| 1024 MB (1 GB) | $0.0000000167 |
| 10240 MB (10 GB) | $0.0000001667 |

**Formula**: Cost = (Memory allocated in GB) × (Duration in seconds) × $0.0000166667 per GB-second

### 3. Provisioned concurrency (if used)

| Metric | Price |
| --- | --- |
| Provisioned concurrency | $0.0000041667 per GB-second |
| Provisioned concurrency requests | $0.0000015 per request |

### Free tier (always free, not just 12 months)

- **1 million requests** per month
- **400,000 GB-seconds** of compute time per month

### Pricing example

A function with **512 MB** memory, running for **200 ms**, invoked **2 million times** per month:

```text
Requests:
  First 1M free
  1M additional × $0.20 / 1M = $0.20

Duration:
  2M invocations × 200 ms = 400,000 seconds
  0.5 GB × 400,000 seconds = 200,000 GB-seconds
  Free tier covers 400,000 GB-seconds → 200,000 is fully covered = $0.00

Total: $0.20 / month
```

### What is NOT billed

- Time spent in cold start initialization (`Init Duration`) is **not billed** for on-demand functions
- Idle time between invocations
- Failed invocations that error before the handler runs

### What IS billed

- Handler execution time (including time waiting on I/O like HTTP calls, DB queries)
- SnapStart restore time (`Restore Duration` is billed)
- Provisioned concurrency idle time (you pay even if the function is not invoked)

---

## Concurrency

Concurrency is the number of function instances processing events at the same time.

### Three types of concurrency

#### 1. Unreserved concurrency

This is the **default** behavior. Your function shares the account-level concurrency pool with all other Lambda functions in the same region.

- Default account limit: **1,000 concurrent executions** (can be increased via support request)
- All functions compete for this shared pool
- No guarantees — one function could consume all available concurrency

```text
Account limit: 1000
Function A uses: 600
Function B uses: 300
Function C uses: 100
→ Pool is full. New invocations will be throttled.
```

#### 2. Reserved concurrency

Reserved concurrency **guarantees** a set number of concurrent instances for a specific function. It also acts as a **maximum limit**.

- The reserved amount is subtracted from the account-level pool
- Other functions cannot use the reserved capacity
- The function cannot exceed its reserved amount either

```text
Account limit: 1000
Function A reserved: 200  → guaranteed 200, max 200
Function B reserved: 100  → guaranteed 100, max 100
Remaining unreserved pool: 700 (shared by all other functions)
```

**When to use reserved concurrency**:

- Protect critical functions from being starved
- Limit a function to prevent it from overwhelming a downstream service (like a database)
- Setting reserved concurrency to **0** effectively disables a function

To configure in the console:

1. Open your Lambda function
2. Go to **Configuration** → **Concurrency**
3. Choose **Reserve concurrency**
4. Enter the number

#### 3. Provisioned concurrency

Provisioned concurrency keeps a specified number of execution environments **initialized and ready** at all times.

- Eliminates cold starts entirely for the provisioned instances
- You pay for provisioned environments even when idle
- Works only with **published versions** or **aliases** (not `$LATEST`)
- **Cannot be combined with SnapStart**

```text
Function A provisioned: 50
→ 50 instances are always warm and ready
→ If traffic exceeds 50, additional instances scale on-demand (with cold starts)
```

**When to use provisioned concurrency**:

- Latency-sensitive APIs where cold starts are unacceptable
- Predictable traffic patterns where you know the baseline load
- Financial or real-time applications

### Concurrency summary

| Type | Guaranteed? | Max limit? | Cold starts? | Cost |
| --- | --- | --- | --- | --- |
| Unreserved | No | Account limit | Yes | Standard |
| Reserved | Yes (up to N) | Yes (exactly N) | Yes | Standard |
| Provisioned | Yes (up to N) | No hard cap (scales beyond) | No (for provisioned instances) | Higher (idle cost) |

---

## Ephemeral storage

Lambda provides temporary disk storage mounted at `/tmp`.

### Defaults and limits

| Setting | Value |
| --- | --- |
| Default size | 512 MB |
| Minimum | 512 MB |
| Maximum | 10,240 MB (10 GB) |
| Path | `/tmp` |

### What ephemeral storage is for

- Downloading and processing large files (images, videos, CSVs)
- Extracting archives
- Temporary caching between handler invocations **within the same execution environment**
- Storing intermediate computation results

### Important behavior

- `/tmp` persists between invocations **only if the same execution environment is reused** (warm start)
- A cold start always gives you a **fresh `/tmp`**
- You are responsible for managing the space — Lambda does not auto-clean `/tmp` between warm invocations
- Anything stored in `/tmp` is **not shared** between concurrent instances

### Configuring ephemeral storage

In the AWS Console:

1. Open your Lambda function
2. Go to **Configuration** → **General configuration**
3. Edit **Ephemeral storage**
4. Set the value (512 MB to 10,240 MB)

In AWS SAM:

```yaml
Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: index.handler
      Runtime: nodejs20.x
      EphemeralStorage:
        Size: 2048  # in MB
```

### Pricing for ephemeral storage

Storage beyond the default 512 MB is billed:

| Metric | Price |
| --- | --- |
| First 512 MB | Free (included) |
| Additional storage | $0.0000000309 per GB-second |

### Example: cleaning up /tmp in warm invocations

```js
import { readdirSync, unlinkSync } from 'fs';
import { join } from 'path';

const cleanTmp = () => {
  const files = readdirSync('/tmp');
  for (const file of files) {
    unlinkSync(join('/tmp', file));
  }
};

export const handler = async (event) => {
  // Clean up from previous invocation
  cleanTmp();

  // Your processing logic here
  // e.g., download a file to /tmp, process it, upload results

  return { statusCode: 200, body: 'Done' };
};
```

---

## Quick reference

| Topic | Key point |
| --- | --- |
| Cold start | New environment setup — adds latency |
| SnapStart | Snapshot-based restore for Java — cuts cold start by ~10x |
| Feral | Scala-first serverless framework (Typelevel) — compile to Scala.js for Node.js cold starts |
| Quarkus | JVM framework with GraalVM native image support — cold starts as low as ~10-50 ms |
| Pricing | Pay per request + duration (GB-seconds) |
| Free tier | 1M requests + 400K GB-seconds per month (always free) |
| Unreserved concurrency | Shared pool, no guarantees |
| Reserved concurrency | Guaranteed and capped for a specific function |
| Provisioned concurrency | Pre-warmed instances, no cold starts, higher cost |
| Ephemeral storage | `/tmp`, 512 MB default, up to 10 GB |
