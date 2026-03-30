# Lambda Configuration

This guide covers all the configuration options available for AWS Lambda functions.

## General Configuration

### Basic Settings
- **Function name** - Unique identifier (1-64 characters, alphanumeric, hyphens, underscores)
- **Runtime** - Execution environment (Node.js, Python, Java, .NET, Go, Ruby, Custom Runtime)
- **Architecture** - x86_64 or arm64 (Graviton)
- **Handler** - Entry point for function execution
- **Memory** - 128 MB to 10,240 MB (1 MB increments)
- **Timeout** - 1 second to 15 minutes
- **Ephemeral storage** - 512 MB to 10,240 MB (for /tmp)

### Runtime Settings
- **Runtime version** - Specific version or automatic updates
- **Handler method** - Function entry point (varies by runtime)
- **Memory allocation** - Directly affects CPU power and pricing

## Triggers

Lambda functions can be invoked by various AWS services:

### API Gateway
- REST API endpoints
- HTTP API endpoints
- WebSocket connections

### Event Sources
- **S3** - Object creation, deletion events
- **SNS** - Topic notifications
- **SQS** - Queue message processing
- **EventBridge** - Scheduled events, custom event patterns
- **DynamoDB Streams** - Table change events
- **Kinesis** - Stream processing
- **Kafka** / **MSK** - Managed streaming

### Direct Invocation
- AWS SDK invoke calls
- Function URL endpoints
- Cross-account invocation

## Permissions

### Execution Role
- **IAM Role** - Required for function execution
- **Resource policies** - Control which services can invoke the function
- **Permissions boundaries** - Limit maximum permissions

### Key Policies
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    }
  ]
}
```

### Common Permissions Needed
- CloudWatch Logs (for logging)
- VPC access (for ENI management)
- Service-specific permissions (S3, DynamoDB, etc.)

### IAM Role
The execution role is an IAM role that Lambda assumes when invoking your function. It grants the function permissions to access AWS services and resources.

**Key characteristics:**
- **Trust policy** - Allows Lambda service (`lambda.amazonaws.com`) to assume the role
- **Permission policy** - Defines what actions the function can perform on which resources
- **Temporary credentials** - Rotated automatically by AWS

### Resource Summary by Action
Common actions Lambda functions need, grouped by AWS service:

| Service | Actions | Purpose |
|---------|---------|---------|
| CloudWatch Logs | `CreateLogGroup`, `CreateLogStream`, `PutLogEvents` | Write function logs |
| X-Ray | `PutTraceSegments`, `PutTelemetryRecords` | Distributed tracing |
| S3 | `GetObject`, `PutObject`, `DeleteObject` | Object storage operations |
| DynamoDB | `GetItem`, `PutItem`, `UpdateItem`, `DeleteItem`, `Query`, `Scan` | Database operations |
| SNS | `Publish` | Send notifications |
| SQS | `SendMessage`, `ReceiveMessage`, `DeleteMessage` | Queue operations |
| Secrets Manager | `GetSecretValue` | Retrieve secrets |
| KMS | `Decrypt`, `GenerateDataKey` | Encryption operations |

### Resource Summary by Resource
View permissions organized by the resource being accessed:

| Resource Type | Example ARN Pattern | Required Permissions |
|---------------|---------------------|---------------------|
| S3 Bucket | `arn:aws:s3:::bucket-name/*` | `s3:GetObject`, `s3:PutObject` |
| DynamoDB Table | `arn:aws:dynamodb:region:account:table/table-name` | `dynamodb:GetItem`, `dynamodb:PutItem` |
| SNS Topic | `arn:aws:sns:region:account:topic-name` | `sns:Publish` |
| SQS Queue | `arn:aws:sqs:region:account:queue-name` | `sqs:SendMessage`, `sqs:ReceiveMessage` |
| Secrets Manager | `arn:aws:secretsmanager:region:account:secret:name-*` | `secretsmanager:GetSecretValue` |
| CloudWatch Logs | `arn:aws:logs:region:account:log-group:/aws/lambda/*` | `logs:CreateLogStream`, `logs:PutLogEvents` |

### Resource-Based Policy Statements
Resource-based policies (also called function policies) control which services and accounts can invoke your Lambda function.

**Example: Allow S3 to invoke function:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3Invoke",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "lambda:InvokeFunction",
      "Resource": "arn:aws:lambda:region:account:function:function-name",
      "Condition": {
        "StringEquals": {
          "AWS:SourceAccount": "123456789012"
        },
        "ArnLike": {
          "AWS:SourceArn": "arn:aws:s3:::bucket-name"
        }
      }
    }
  ]
}
```

**Example: Allow API Gateway to invoke function:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAPIGatewayInvoke",
      "Effect": "Allow",
      "Principal": {
        "Service": "apigateway.amazonaws.com"
      },
      "Action": "lambda:InvokeFunction",
      "Resource": "arn:aws:lambda:region:account:function:function-name",
      "Condition": {
        "ArnLike": {
          "AWS:SourceArn": "arn:aws:execute-api:region:account:api-id/*/GET/resource"
        }
      }
    }
  ]
}
```

**Example: Cross-account access:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCrossAccountInvoke",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111122223333:root"
      },
      "Action": "lambda:InvokeFunction",
      "Resource": "arn:aws:lambda:region:account:function:function-name"
    }
  ]
}
```

## Destinations

Configure what happens to invocation records after execution:

### Asynchronous Invocation Destinations
- **On Success** - Send successful invocation records to:
  - SQS queue
  - SNS topic
  - EventBridge event bus
  - Lambda function

- **On Failure** - Send failed invocation records (including error details) to:
  - SQS queue
  - SNS topic
  - EventBridge event bus
  - Lambda function

### Use Cases
- **Event-driven workflows** - Chain functions together
- **Error handling** - Route failures to dead-letter queues
- **Audit logging** - Record all invocations for compliance

## Function URL

Create a dedicated HTTPS endpoint for your Lambda function:

### Features
- **Built-in HTTPS** - No API Gateway required
- **CORS support** - Configure cross-origin requests
- **IAM authentication** - Optional AWS signature authentication
- **Invoke mode** - Buffered or response streaming

### Security Options
- **NONE** - Public access (IAM policy still required)
- **AWS_IAM** - Requires valid AWS credentials

### When to Use
- Simple HTTP endpoints
- Webhook receivers
- Streaming responses (response streaming mode)

## Environment Variables

Key-value pairs accessible within your function:

### Common Use Cases
- **Database connection strings** - Without hardcoding credentials
- **Feature flags** - Toggle functionality without code changes
- **Stage-specific settings** - Dev/staging/prod configurations
- **Third-party API keys** - Securely store external service credentials

### Security
- **Encryption at rest** - AWS KMS encryption
- **Encryption in transit** - Automatic HTTPS
- **Secrets Manager** - For sensitive values, use `GetSecretValue` calls

### Access in Code
```python
import os

db_host = os.environ['DB_HOST']
api_key = os.environ['API_KEY']
```

## Tags

Metadata labels for organization and cost tracking:

### Use Cases
- **Cost allocation** - Track spending by team/project
- **Resource organization** - Group related functions
- **Access control** - IAM policies based on tags
- **Automation** - Trigger workflows based on tags

### Common Tag Keys
- `Environment` - dev, staging, production
- `Project` - Application or service name
- `Owner` - Team or individual responsible
- `CostCenter` - Billing code

## VPC

Connect Lambda to your Virtual Private Cloud:

### Configuration Requirements
- **VPC ID** - Target VPC
- **Subnets** - Private subnets with NAT Gateway for internet access
- **Security groups** - Control inbound/outbound traffic

### Internet Access from VPC
Lambda functions in a VPC require a **NAT Gateway** or **VPC Endpoints** for:
- Accessing public AWS services (S3, DynamoDB)
- Calling external APIs
- Downloading dependencies

### ENI (Elastic Network Interface)
- Lambda creates ENIs for VPC access
- Scales automatically based on concurrency
- Cold start impact due to ENI creation

### Best Practices
- Use VPC only when necessary (database access, private resources)
- Use VPC endpoints for AWS service access
- Consider VPC-Lambda integration latency

## RDS Databases

Connect to Amazon RDS and Aurora databases:

### Connection Methods
- **Direct connection** - Standard database driver
- **RDS Proxy** - Connection pooling for better performance
- **Data API** - HTTP-based access (Aurora Serverless v1/v2)

### RDS Proxy Benefits
- **Connection pooling** - Reduces database load
- **Failover handling** - Automatic reconnect on failovers
- **IAM authentication** - Secure password-less access

### Configuration
- VPC required (same as RDS instance)
- Security group rules for database port access
- Connection string with proxy endpoint

## Monitoring and Operations Tools

### CloudWatch Logs
- Automatic log collection to CloudWatch Logs
- Log retention configuration (1 day to 10 years)
- Log groups named `/aws/lambda/<function-name>`

### CloudWatch Metrics
- **Invocations** - Total requests
- **Duration** - Execution time
- **Errors** - Failed invocations
- **Throttles** - Rate limit hits
- **ConcurrentExecutions** - Simultaneous invocations

### X-Ray Tracing
- **Active tracing** - Automatic request tracing
- **Service map** - Visualize service dependencies
- **Subsegments** - Custom tracing in code

### CloudWatch Alarms
- Set thresholds on metrics
- SNS notifications for alerts
- Automated responses (Lambda, EC2 actions)

### Lambda Insights
- Enhanced monitoring with curated dashboards
- Automatic performance issue detection
- Pre-built operational metrics

## Concurrency and Recursion Detection



### Recursion Detection
- **Automatic protection** - Detects and stops recursive invocation loops
- **SQS, S3, DynamoDB triggers** - Monitors for circular patterns
- **Prevents runaway costs** - Stops after detecting loops


## Asynchronous Invocation

Configure how Lambda handles events from event sources:

### Retry Behavior
- **2 retry attempts** - Automatic retries on failure
- **Exponential backoff** - Increasing delay between retries
- **Maximum age** - Events expire after 6 hours by default

### Dead Letter Queue (DLQ)
- **SQS or SNS** - Store failed events after retries exhausted
- **Error handling** - Process failed events separately
- **Monitoring** - Track DLQ depth for alerting

### Event Filtering
- **Filter criteria** - Process only matching events
- **Reduce invocations** - Filter at the source
- **Cost optimization** - Pay only for relevant events

### Batching
- **Batch size** - Process multiple records per invocation
- **Batch window** - Wait time to gather records
- **Parallelization** - Concurrent batches for high throughput

## Code Signing

Code signing cryptographically verifies that your Lambda function code comes from a trusted source and has not been tampered with since publication.

### How Code Signing Works

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│  Developer  │────▶│  AWS Signer  │────▶│  Signed ZIP/JAR │
│  (Publisher)│     │  (Signing)   │     │  (with signature)│
└─────────────┘     └──────────────┘     └─────────────────┘
                                                  │
                                                  ▼
┌─────────────────────────────────────────────────────────┐
│                    Lambda Service                        │
│  ┌─────────────┐    ┌─────────────┐    ┌────────────┐  │
│  │  Extract    │───▶│  Validate   │───▶│  Deploy    │  │
│  │  Signature  │    │  Signature  │    │  Function  │  │
│  └─────────────┘    └─────────────┘    └────────────┘  │
│                           │                              │
│                           ▼ (fails)                       │
│                    ┌─────────────┐                        │
│                    │   Block     │                        │
│                    │  Deployment │                        │
│                    └─────────────┘                        │
└─────────────────────────────────────────────────────────┘
```

**The workflow:**
1. **Package** - Developer creates deployment package (ZIP or JAR)
2. **Sign** - AWS Signer attaches cryptographic signature using signing profile
3. **Upload** - Signed package stored in S3
4. **Validate** - Lambda validates signature matches trusted publisher before deployment
5. **Deploy** - Only valid code gets deployed; tampered code is rejected

### Step-by-Step Setup

**Step 1: Create a Signing Profile**
```bash
aws signer put-signing-profile \
    --profile-name MySigningProfile \
    --platform AWSLambda-SHA384-ECDSA \
    --signing-material file://signing-material.json
```

**Step 2: Enable Code Signing on Lambda Function**
```bash
aws lambda create-code-signing-config \
    --description "My signing config" \
    --allowed-publishers SigningProfileVersionArns=arn:aws:signer:region:account:/signing-profiles/MySigningProfile/1 \
    --code-signing-policies UntrustedArtifactOnDeployment=Enforce
```

**Step 3: Sign Your Code Package**
```bash
aws signer sign-object \
    --profile-name MySigningProfile \
    --source s3://bucket/unsigned-code.zip \
    --destination s3://bucket/signed-code.zip
```

**Step 4: Attach Config to Lambda Function**
```bash
aws lambda update-function-code \
    --function-name my-function \
    --s3-bucket bucket \
    --s3-key signed-code.zip
```

### AWS Signer Components

| Component | Purpose |
|-----------|---------|
| **Signing Profile** | Defines signing algorithm and certificate. Think of it as your "publisher identity." |
| **Signing Job** | The actual signing operation that produces a signature file. |
| **Signature File** | Cryptographic proof attached to your code package. |
| **Validation** | Lambda checks the signature against allowed publishers before deployment. |

### Validation Modes

| Mode | Behavior | Use Case |
|------|----------|----------|
| **Enforce** | Block deployment if signature is invalid or from untrusted publisher | Production environments |
| **Warn** | Allow deployment but log warning if signature issues detected | Development/testing transition |

### Allowed Publishers

Control which signing profiles can deploy to your function:

```json
{
  "AllowedPublishers": {
    "SigningProfileVersionArns": [
      "arn:aws:signer:us-east-1:123456789012:/signing-profiles/TeamA/1",
      "arn:aws:signer:us-east-1:123456789012:/signing-profiles/TeamB/2"
    ]
  }
}
```

### Benefits

| Benefit | Explanation |
|---------|-------------|
| **Tamper Detection** | Any modification to code after signing invalidates the signature. Lambda rejects tampered packages. |
| **Publisher Verification** | Only code signed by approved profiles can be deployed. Prevents unauthorized deployments. |
| **Compliance** | Meets security standards (SOC 2, PCI-DSS) requiring code integrity checks. |
| **Supply Chain Security** | Verifies code provenance from developer to production deployment. |
| **Audit Trail** | AWS CloudTrail logs every signing and validation event. |

### Supported Runtimes

Code signing works with:
- **ZIP packages** - Python, Node.js, Ruby, Go, .NET
- **JAR files** - Java functions
- **Container images** - Lambda container image functions (requires signing container layers)

### When Code Signing Fails

Lambda blocks deployment when:
- Signature is missing from the package
- Signature is corrupted or malformed
- Signing profile is not in the allowed publishers list
- Package was modified after signing
- Signing certificate has expired
- Signature algorithm is not supported

## File Systems

Mount EFS (Elastic File System) to Lambda:

### Use Cases
- **Shared storage** - Multiple functions access same data
- **Large files** - Files larger than 512 MB /tmp limit
- **Persistent storage** - Data persists across invocations
- **Machine learning** - Access large models or datasets

### Configuration
- **Access point** - EFS access point for permissions
- **Mount path** - Local path in function (e.g., `/mnt/data`)
- **VPC required** - EFS and Lambda must be in same VPC

### Performance
- **Bursting throughput** - Scales automatically
- **Provisioned throughput** - Guaranteed performance
- **Latency** - Higher than local /tmp storage

## State Machines

Integration with AWS Step Functions:

### Step Functions Workflow
- **Orchestration** - Coordinate multiple Lambda functions
- **State machines** - Visual workflow design
- **Error handling** - Built-in retry and catch logic

### Integration Patterns
- **Standard workflows** - Exactly-once execution, up to 1 year
- **Express workflows** - High throughput, up to 5 minutes
- **Synchronous** - Wait for completion
- **Asynchronous** - Fire-and-forget

### Use Cases
- **Long-running processes** - Beyond Lambda's 15-minute limit
- **Human approval workflows** - Wait for external signals
- **Complex error handling** - Try-catch-finally patterns
- **Parallel processing** - Map states for concurrent execution

## Configuration Best Practices

1. **Least privilege** - Grant minimum required permissions
2. **Environment-specific configs** - Use environment variables for stage differences
3. **Monitor concurrency** - Set appropriate limits and alarms
4. **Enable X-Ray** - For distributed tracing in multi-service architectures
5. **Use VPC endpoints** - Reduce latency and improve security
6. **Tag everything** - Enable cost tracking and organization
7. **Secure secrets** - Use Secrets Manager, not environment variables for sensitive data
8. **Optimize memory** - Balance performance and cost
