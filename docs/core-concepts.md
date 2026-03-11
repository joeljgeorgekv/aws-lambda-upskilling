# Core Concepts

## The handler

The handler is the main function AWS Lambda calls when your function is invoked.

Think of it as the entry point.

### Node.js example

```js
exports.handler = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify({ message: 'Hello from Lambda' })
  };
};
```

### Python example

```python
def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": "Hello from Lambda"
    }
```

## Event object

The `event` contains the data that triggered the function.

Examples:

- HTTP request details from API Gateway
- File information from S3
- Message payload from SQS

Understanding the event structure is one of the most important Lambda skills.

## Context object

The `context` provides runtime information about the invocation.

It can include:

- Function name
- Remaining execution time
- Request ID
- Memory limit

Beginners often ignore `context` at first, and that is fine.

## Timeout

Timeout is the maximum time your function is allowed to run.

If your function takes too long, Lambda stops it.

Beginner advice:

- Start with a reasonable timeout
- Do not set it too high without a reason
- Investigate why a function is slow instead of only increasing timeout

## Memory

You choose how much memory your function gets.

More memory can also improve CPU allocation.

Important idea:

- Too little memory can make functions slow
- Too much memory can increase cost unnecessarily

## IAM execution role

Your Lambda function needs permissions to do things like:

- Write logs
- Read from S3
- Send messages to SQS
- Access DynamoDB

This is handled with an IAM role.

Beginner rule:

- Grant only the permissions your function actually needs

## Environment variables

Environment variables let you store configuration outside your code.

Examples:

- Table names
- API base URLs
- Feature flags

Do not store secrets casually in plain code.

## Concurrency

Concurrency means how many instances of your function can run at the same time.

This matters when:

- Traffic spikes
- Many events arrive together
- Downstream services have rate limits

As a beginner, just remember:

- Lambda scales automatically
- But scaling still needs thoughtful design

## Cold starts

A cold start happens when Lambda creates a new execution environment.

You may notice:

- Slight delay on some invocations
- More effect in some languages or larger packages

Do not panic about cold starts too early.
Understand them, but first learn the basics well.

## Continue next

Move to [First Lambda](./first-lambda.md).
