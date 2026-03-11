# Integrations

## Why integrations matter

Lambda becomes powerful when it reacts to other AWS services.

Instead of running by itself, it is often triggered by:

- API Gateway
- S3
- EventBridge
- SQS

## API Gateway

Use API Gateway when you want Lambda to respond to HTTP requests.

Examples:

- `GET /health`
- `POST /users`
- `DELETE /tasks/123`

This is one of the most common Lambda use cases.

### Good for beginners because

- It feels familiar if you know web development
- You can build APIs quickly
- You learn request and response patterns

## S3

S3 can trigger Lambda when file events happen.

Examples:

- A file is uploaded
- A file is deleted
- A new image needs processing

### Common beginner use cases

- Resize uploaded images
- Validate file names
- Extract metadata

## EventBridge

EventBridge is useful for scheduled or event-driven workflows.

Examples:

- Run every morning at 8 AM
- React to application events
- Trigger automation pipelines

### Good beginner examples

- Send a daily summary
- Clean temporary data on a schedule
- Run a periodic report

## SQS

SQS lets Lambda process messages from a queue.

This is useful when:

- Work should happen asynchronously
- Systems should be decoupled
- You want retry support

### Beginner example

An app sends an order message to a queue, and Lambda processes it later.

## How to choose the right trigger

Ask yourself:

- Is this request-response? Use API Gateway.
- Is this file-based? Use S3.
- Is this scheduled or event-bus based? Use EventBridge.
- Is this queue-based background work? Use SQS.

## Important beginner lesson

Different triggers produce different event payloads.

That means:

- Your function code must understand the specific event format
- Debugging often starts by printing the received event

## Simple debugging tip

When learning a new trigger, temporarily log the full event:

### Node.js

```js
console.log(JSON.stringify(event, null, 2));
```

### Python

```python
import json
print(json.dumps(event, indent=2))
```

This helps you understand what Lambda actually receives.

## Continue next

Move to [Best Practices](#/best-practices).
