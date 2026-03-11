# AWS Lambda Upskilling

> A beginner-friendly roadmap to learn, build, and deploy serverless functions with AWS Lambda.

[Get Started](#/foundations) [GitHub Pages Setup](#/github-pages)

![color](https://img.shields.io/badge/Level-Beginner-success)
![aws](https://img.shields.io/badge/Focus-AWS%20Lambda-orange)
![serverless](https://img.shields.io/badge/Style-Hands%20On-blue)

## What you will learn

- What AWS Lambda is and when to use it
- Core serverless concepts without too much jargon
- How Lambda works with events, permissions, and logs
- How to build your first functions using Node.js or Python
- How to connect Lambda with API Gateway, S3, and EventBridge
- How to deploy and host your documentation with GitHub Pages

## Who this is for

This guide is for you if:

- You are new to AWS
- You want a structured Lambda learning path
- You prefer simple explanations and small practical steps
- You want to build confidence before moving into advanced serverless topics

## Learning path

### 1. Foundations

Start here if AWS Lambda is completely new to you.

- Learn what serverless means in practice
- Understand functions, triggers, and execution flow
- Get familiar with AWS regions, IAM, and CloudWatch at a high level

[Open Foundations](#/foundations)

### 2. Core AWS Lambda Concepts

Learn the moving parts that matter most.

- Function handler
- Runtime
- Events and context
- Memory, timeout, and concurrency
- IAM execution roles
- Environment variables

[Open Core Concepts](#/core-concepts)

### 3. Build Your First Lambda

Create a simple function and test it.

- Console-based hello world
- Local folder structure
- Sample Node.js and Python handlers
- Test events
- Viewing logs in CloudWatch

[Open First Lambda](#/first-lambda)

### 4. Integrations

See how Lambda connects with other AWS services.

- API Gateway for HTTP APIs
- S3 for file-based triggers
- EventBridge for event-driven systems
- SQS for queue processing

[Open Integrations](#/integrations)

### 5. Best Practices

Develop good habits early.

- Keep functions small and focused
- Use least-privilege IAM permissions
- Handle retries and failures carefully
- Monitor logs and metrics
- Avoid putting secrets directly in code

[Open Best Practices](#/best-practices)

### 6. Mini Project

Build a simple beginner project to apply what you learned.

- Create an API endpoint using API Gateway + Lambda
- Validate input
- Return JSON responses
- Review logs and improve the function

[Open Mini Project](#/mini-project)

## Suggested 2-week plan

### Week 1

- Day 1: Read Foundations
- Day 2: Read Core Concepts
- Day 3: Build your first Lambda in the console
- Day 4: Add logs and test different inputs
- Day 5: Learn API Gateway basics
- Day 6: Try an S3 trigger example
- Day 7: Review what confused you and repeat the tricky parts

### Week 2

- Day 8: Read Best Practices
- Day 9: Build the mini project
- Day 10: Add error handling
- Day 11: Add environment variables
- Day 12: Explore IAM permissions
- Day 13: Practice deployment flow
- Day 14: Publish your notes with GitHub Pages

## Modules

- [Foundations](foundations.md)
- [Core Concepts](core-concepts.md)
- [First Lambda](first-lambda.md)
- [Integrations](integrations.md)
- [Best Practices](best-practices.md)
- [Mini Project](mini-project.md)
- [GitHub Pages Hosting](github-pages.md)

## Quick glossary

- **Lambda**: A service that runs your code without managing servers.
- **Serverless**: You write code, and the cloud provider handles server management.
- **Trigger**: An event source that invokes your Lambda function.
- **IAM Role**: The permissions your function uses to access AWS services.
- **CloudWatch Logs**: Where your Lambda execution logs appear.
- **Cold Start**: Extra startup time when a new Lambda execution environment is created.

## How to use this guide

- Read the sections in order if you are a beginner
- Open AWS and try small tasks as you learn
- Take notes in your own words
- Repeat the build steps instead of only reading
- Focus on understanding events and permissions early

## What to learn next after this

Once you finish this guide, you can move into:

- AWS SAM or Serverless Framework
- Lambda layers
- VPC networking
- Secrets Manager and Parameter Store
- CI/CD for serverless deployments
- Observability with CloudWatch metrics and alarms
