# Foundations

## What is AWS Lambda?

AWS Lambda is a compute service that lets you run code without managing servers.
AWS lambda is a Function as a Service (FaaS) platform.

That means:
- You write the function
- AWS runs it for you
- You pay only when it runs
- You do not need to manage operating systems or servers manually

## What does serverless really mean?

Serverless does **not** mean there are no servers.
It means **you do not manage the servers yourself**.

AWS handles:
- Provisioning
- Scaling
- Availability
- Infrastructure maintenance

## When should you use Lambda?

Lambda is a good fit when:
- You need small units of code triggered by events
- Traffic is unpredictable
- You want quick deployment for lightweight backend logic
- You are building APIs, automations, or integrations

Common use cases:
- API backends
- File processing
- Scheduled jobs
- Notification workflows
- Event-driven applications

## When Lambda may not be the best choice

Lambda may not be ideal when:

- Your workload runs for a long time continuously
- You need full control over the server environment
- You have very large monolithic applications
- You need very low-latency warm performance at all times

## Core beginner concepts

Before building with Lambda, understand these ideas:

### Function

A function is the code you want AWS Lambda to run.

### Trigger

A trigger is the event that starts your function.
Examples:

- HTTP request from API Gateway
- File upload to S3
- Message in SQS
- Scheduled event from EventBridge


### Execution role

This is the IAM role that gives your Lambda permission to use AWS services.

### Logs

When your function runs, logs are sent to CloudWatch Logs.
This is one of the most useful tools for debugging.

## Basic Lambda flow

A simple Lambda flow looks like this:

1. An event happens
2. AWS Lambda receives the event
3. Your function code runs
4. The function returns a result or performs an action
5. Logs and metrics are recorded

## Why beginners like Lambda

- You can build real things quickly
- You do not need to learn infrastructure first
- It works well with many AWS services
- It teaches cloud, events, permissions, and observability together

## Free tier

AWS Lambda includes a generous free tier:

- **1,000,000 requests** per month
- **400,000 GB-seconds** of compute time per month

This is enough for many small projects and learning experiments.

## Pricing

After the free tier:

- **$0.20 per 1 million requests**
- **$1 per 600,000 GB-seconds**

GB-seconds = memory in GB × execution time in seconds

## Limits

Important limits to know:

- **Memory allocation**: 128 MB to 10 GB per function
- **Maximum execution time**: 15 minutes (900 seconds)
- **Payload size**: 6 MB for synchronous, 256 KB for asynchronous

## Supported runtimes

Lambda supports many languages:

- Node.js
- Python
- Java
- C# (.NET Core, PowerShell)
- Ruby
- Go (via custom runtime)
- Rust (via custom runtime)
- Custom runtime API (for any language)

## Container images

You can also deploy Lambda functions as container images.

The container must implement the Lambda Runtime API.

## Common integrations

Lambda integrates with many AWS services:

1. API Gateway
2. Kinesis
3. DynamoDB
4. S3
5. CloudFront
6. EventBridge
7. CloudWatch Logs
8. SNS
9. SQS
10. Cognito


## Continue next

Move to [Core Concepts](./core-concepts.md).
