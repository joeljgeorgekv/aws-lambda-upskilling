# First Lambda

## Goal

In this section, you will build two beginner-friendly Lambda examples that both add two numbers:

- A **synchronous Lambda** using **API Gateway**
- An **asynchronous Lambda** using **SQS**

This helps you understand one of the most important Lambda ideas:

- **Synchronous** means the caller waits for a response
- **Asynchronous** means the event is sent and processed later

## Case 1: Synchronous Lambda with API Gateway

In this case:

- A client sends two numbers in an HTTP request
- API Gateway passes the request to Lambda
- Lambda adds the two numbers
- Lambda returns the result immediately

### Flow

```text
Client -> API Gateway -> Lambda -> Response
```

### Step 1: Create the Lambda function

1. Sign in to AWS
2. Open the Lambda service
3. Click **Create function**
4. Choose **Author from scratch**
5. Enter a function name like `add-two-numbers-sync`
6. Choose **Node.js** as the runtime
7. Create or use a basic execution role
8. Click **Create function**

### Step 2: Add the Lambda code

Use this Node.js example:

```js
export const handler = async (event) => {
  const params = event.queryStringParameters || {};
  const a = Number(params.a);
  const b = Number(params.b);

  if (Number.isNaN(a) || Number.isNaN(b)) {
    return {
      statusCode: 400,
      body: JSON.stringify({
        message: 'Please provide valid numbers for a and b'
      })
    };
  }

  return {
    statusCode: 200,
    body: JSON.stringify({
      a,
      b,
      sum: a + b
    })
  };
};
```

### Step 3: Create the API Gateway trigger

1. Open your Lambda function
2. Click **Add trigger**
3. Select **API Gateway**
4. Create a new API
5. Choose **HTTP API**
6. Set security as needed for learning
7. Add the trigger

### Step 4: Test with query parameters

Call the endpoint with two numbers:

```text
https://your-api-url?a=10&b=20
```

Expected response:

```json
{
  "a": 10,
  "b": 20,
  "sum": 30
}
```

### Step 5: Understand the event

For API Gateway, Lambda receives request information inside the `event` object.

In this example, the numbers come from:

```js
event.queryStringParameters
```

That is why the code reads:

```js
const params = event.queryStringParameters || {};
```

### What makes this synchronous?

It is synchronous because the client waits for Lambda to finish and return the result.

That means:

- The caller gets the answer immediately
- Lambda must return a proper HTTP-style response
- This pattern is common for APIs

## Case 2: Asynchronous Lambda with SQS

In this case:

- A message containing two numbers is sent to SQS
- SQS triggers Lambda
- Lambda reads the message and adds the numbers
- The result is processed in the background

### Flow

```text
Producer -> SQS Queue -> Lambda
```

### Step 1: Create the Lambda function

1. Open the Lambda service
2. Click **Create function**
3. Choose **Author from scratch**
4. Enter a function name like `add-two-numbers-async`
5. Choose **Node.js** as the runtime
6. Create or use a role with permissions for SQS and logs  
   **Tip**: Use the managed policy `AWSLambdaSQSQueueExecutionRole` for SQS-triggered functions
7. Click **Create function**

### Step 2: Add the Lambda code

Use this Node.js example:

```js
export const handler = async (event) => {
  for (const record of event.Records) {
    try {
      const body = JSON.parse(record.body);
      const a = Number(body.a);
      const b = Number(body.b);

      if (Number.isNaN(a) || Number.isNaN(b)) {
        console.log("Invalid numbers:", record.body);
        continue;
      }

      const sum = a + b;
      console.log("sum =", sum);
    } catch (err) {
      console.error("Failed to process record:", err, record.body);
    }
  }
};
```

### Step 3: Create an SQS queue

1. Open the SQS service
2. Click **Create queue**
3. Choose **Standard queue**
4. Enter a queue name like `add-two-numbers-queue`
5. Create the queue

### Step 4: Add SQS as the trigger

1. Go back to the Lambda function
2. Click **Add trigger**
3. Select **SQS**
4. Choose `add-two-numbers-queue`
5. Add the trigger

### Step 5: Send a test message to the queue

Use a message body like this:

```json
{
  "a": 5,
  "b": 7
}
```

### Step 6: Check the Lambda logs

Since this is asynchronous, you will not get a direct HTTP response.

Instead, Lambda will process the message in the background.

1. Open the Lambda function
2. Go to **Monitor**
3. Open **CloudWatch Logs**
4. Find the log output for the message

Expected log output:

```text
sum = 12
```

### Step 7: Understand the event

For SQS, Lambda receives an event with a `Records` array.

Each record contains the message body:

```js
record.body
```

That is why the code loops through records and parses each message.

### What makes this asynchronous?

It is asynchronous because the sender does not wait for Lambda to return a direct answer.

That means:

- The message is placed in a queue
- Lambda processes it later
- The result is handled in the background
- This pattern is common for background work

## Synchronous vs asynchronous summary

| Type | Trigger | Input source | Output style |
| --- | --- | --- | --- |
| Synchronous | API Gateway | Query parameters or request body | Direct HTTP response |
| Asynchronous | SQS | Queue message body | Background processing and logs |

## What to look for

- Did the Lambda function receive the event in the expected shape?
- Did the numbers get converted correctly?
- Did the sum logic work?
- Did the logs show the result?
- Did you understand why one case returns immediately and the other does not?

## Common beginner mistakes

### Wrong event shape

API Gateway and SQS send different event formats.

If you write code for the wrong event structure, the function will fail.

### Forgetting to convert strings to numbers

Values from query parameters and message bodies are often strings.

Use `Number(...)` before adding them.

### Missing permissions

If the Lambda role cannot read from the queue or write logs, the function will not work correctly.

### Testing only one path

Always test:

- Valid numbers
- Missing numbers
- Invalid input like text values



