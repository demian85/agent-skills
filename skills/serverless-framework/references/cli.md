# CLI Reference

Complete reference for Serverless Framework CLI commands.

## Deployment Commands

### `serverless deploy` (or `sls deploy`)

Deploys the entire service to AWS.

```bash
# Basic deployment
serverless deploy

# With stage and region
serverless deploy --stage prod --region eu-west-1

# With profile
serverless deploy --aws-profile production

# Force deployment (skip hash check)
serverless deploy --force

# With verbose output
serverless deploy --verbose

# Using changesets
serverless deploy --conceal
```

Options:
- `--stage, -s`: Stage to deploy to
- `--region, -r`: AWS region
- `--aws-profile`: AWS profile to use
- `--force`: Force deployment
- `--verbose, -v`: Verbose output
- `--conceal`: Hide secrets in output
- `--package`: Path to pre-built package
- `--param`: Override parameters (can be used multiple times)

### `serverless deploy function`

Deploys a single function (faster than full deploy).

```bash
serverless deploy function --function myFunction
serverless deploy function -f myFunction --stage prod
```

### `serverless package`

Packages the service without deploying.

```bash
serverless package
serverless package --package /path/to/output
```

### `serverless remove`

Removes the entire service from AWS.

```bash
serverless remove
serverless remove --stage prod --region eu-west-1
```

### `serverless deploy list`

Lists previous deployments.

```bash
serverless deploy list
serverless deploy list --stage prod
```

### `serverless rollback`

Rolls back to a previous deployment.

```bash
serverless rollback --timestamp 1234567890123
```

### `serverless rollback function`

Rolls back a single function.

```bash
serverless rollback function --function myFunction --timestamp 1234567890123
```

## Development Commands

### `serverless dev`

Starts a local development server with live reloading.

```bash
serverless dev
serverless dev --function myFunction
```

Note: Currently only supports Node.js. Watches for file changes and automatically redeploys functions.

### `serverless invoke`

Invokes a deployed function.

```bash
# Invoke with default event
serverless invoke --function myFunction

# Invoke with custom data
serverless invoke -f myFunction --data '{"key":"value"}'

# Invoke with path to event JSON
serverless invoke -f myFunction --path event.json

# Invoke with logs
serverless invoke -f myFunction -l

# Invoke with specific stage
serverless invoke -f myFunction --stage prod
```

### `serverless invoke local`

Invokes a function locally.

```bash
serverless invoke local --function myFunction
serverless invoke local -f myFunction --path event.json
serverless invoke local -f myFunction --data '{"key":"value"}'

# With environment variables
serverless invoke local -f myFunction --env VAR_NAME=value
```

### `serverless logs`

Streams logs from CloudWatch.

```bash
# View logs
serverless logs --function myFunction

# Tail logs (follow)
serverless logs --function myFunction --tail

# Filter logs
serverless logs --function myFunction --filter "ERROR"

# Time range
serverless logs --function myFunction --startTime "2024-01-01T00:00:00" --endTime "2024-01-02T00:00:00"
```

### `serverless metrics`

Displays CloudWatch metrics.

```bash
serverless metrics
serverless metrics --function myFunction
serverless metrics --startTime 2024-01-01 --endTime 2024-01-02
```

## Info & Configuration Commands

### `serverless info`

Displays information about the service.

```bash
serverless info
serverless info --stage prod --verbose
```

### `serverless print`

Prints the processed serverless.yml (with resolved variables).

```bash
serverless print
serverless print --format yaml
serverless print --format json
serverless print --path provider.environment
```

### `serverless plugin`

Manages plugins.

```bash
# Install a plugin
serverless plugin install --name serverless-offline
serverless plugin install -n serverless-esbuild

# Uninstall a plugin
serverless plugin uninstall --name serverless-offline

# List installed plugins
serverless plugin list

# Search for plugins
serverless plugin search --query offline
```

## Utility Commands

### `serverless login`

Logs in to Serverless Framework Dashboard.

```bash
serverless login
```

### `serverless logout`

Logs out from Serverless Framework Dashboard.

```bash
serverless logout
```

### `serverless generate-event`

Generates mock events for testing.

```bash
serverless generate-event --type aws:apiGateway
serverless generate-event --type aws:sns
```

### `serverless config`

Manages configuration.

```bash
# Get/set credentials
serverless config credentials --provider aws --key AKIA... --secret secret...

# Tab completion
serverless config tabcompletion install
serverless config tabcompletion uninstall
```

### `serverless support`

Generates support information.

```bash
serverless support
```

## Compose Commands

When using `serverless-compose.yml`, these commands work at the root level:

```bash
# Deploy all services
serverless deploy

# Deploy specific service
serverless deploy --service=api
serverless api deploy

# Info for all services
serverless info

# Info for specific service
serverless api info

# Remove all services
serverless remove

# Remove specific service
serverless api remove

# Logs for function in specific service
serverless api logs --function myFunction

# Any command for specific service
serverless api invoke --function myFunction
```

## Global Options

Available on all commands:

- `--help, -h`: Show help
- `--version, -v`: Show version
- `--stage, -s`: Stage
- `--region, -r`: Region
- `--config`: Path to config file
- `--aws-profile`: AWS profile
- `--org`: Serverless Framework org
- `--app`: Serverless Framework app
- `--verbose, -v`: Verbose output
- `--debug, -d`: Debug mode
- `--no-color`: Disable color output

## Environment Variables

The CLI respects these environment variables:

- `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY`: AWS credentials
- `AWS_PROFILE`: Default AWS profile
- `AWS_REGION`: Default AWS region
- `SERVERLESS_ACCESS_KEY`: Serverless Framework access key
- `SLS_DEBUG`: Enable debug mode (`*` for all)
- `SLS_WARNING_DISABLE`: Disable specific warnings
- `FORCE_COLOR`: Force color output
