# Best Practices

## Keep functions small

A Lambda function should usually do one clear job.

Good examples:

- Validate a request
- Process a file
- Send a notification
- Transform an event

Avoid making one function handle too many unrelated responsibilities.

## Use least-privilege IAM

Only give the function the permissions it actually needs.

Bad habit:

- Giving full access to many AWS services just to make things work quickly

Better habit:

- Start with the minimum required permissions
- Add only what is necessary

## Log useful information

Logs should help you debug without exposing sensitive data.

Useful things to log:

- Request IDs
- Key execution steps
- Error messages
- Relevant high-level input details

Avoid logging:

- Passwords
- Secrets
- Full personal data unnecessarily

## Handle errors clearly

Do not let your function fail silently.

Instead:

- Validate input
- Catch expected errors
- Return helpful messages
- Write useful logs

## Use environment variables for configuration

Avoid hardcoding values that change across environments.

Examples:

- Table names
- Bucket names
- API URLs

## Watch timeout and memory

If your function is slow or unstable:

- Check timeout
- Check memory
- Check package size
- Check network calls

Do not blindly increase resource settings without understanding the cause.

## Make code idempotent when possible

Idempotent means repeated execution should not create incorrect duplicate effects.

This matters because some event sources may retry.

## Keep dependencies small

Large deployment packages can slow things down and make maintenance harder.

As a beginner:

- Use only the libraries you truly need
- Prefer simple code when possible

## Separate dev and prod thinking

Even in small projects, try to think about:

- Different environments
- Different config values
- Safer deployment habits

## Best-practice checklist

Before calling your Lambda function ready, ask:

- Is it doing one main job?
- Does it have only the permissions it needs?
- Does it log useful information?
- Does it handle invalid input?
- Are config values kept outside code when appropriate?

## Continue next

Move to [Mini Project](#/mini-project).
