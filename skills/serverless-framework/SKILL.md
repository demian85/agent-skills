---
name: serverless-framework
description: >
  Comprehensive Serverless Framework skill for AWS Lambda, API Gateway, and serverless application development.
  Use when working with serverless.yml configuration, Serverless Framework CLI commands, deploying Lambda functions,
  configuring API Gateway (REST/HTTP/Websocket), setting up event triggers (S3, SQS, SNS, DynamoDB Streams, Schedule),
  managing serverless deployments, troubleshooting serverless applications, or integrating with the Serverless MCP Server.
  Covers serverless architecture patterns, IaC best practices, CI/CD workflows, and AWS serverless service integrations.
---

# Serverless Framework

The Serverless Framework is an Infrastructure as Code (IaC) tool for building and deploying serverless applications on AWS. It abstracts CloudFormation into a developer-friendly YAML configuration (`serverless.yml`) while providing CLI commands for the entire lifecycle.

## When to Use This Skill

- Writing or editing `serverless.yml` configurations
- Deploying or managing AWS Lambda functions
- Configuring API Gateway (REST v1, HTTP v2, WebSocket)
- Setting up event triggers and integrations
- Troubleshooting serverless application issues
- Working with Serverless Framework Compose for multi-service projects
- Using the Serverless MCP Server for AI-assisted debugging

## Core Concepts

### Service Structure

```
my-service/
├── serverless.yml          # Main configuration
├── src/
│   └── handler.js          # Function code
├── package.json
└── .env                     # Environment variables (not committed)
```

### Minimal serverless.yml

```yaml
service: my-service

provider:
  name: aws
  runtime: nodejs20.x
  stage: ${opt:stage, 'dev'}
  region: ${opt:region, 'us-east-1'}

functions:
  hello:
    handler: src/handler.hello
    events:
      - httpApi:
          path: /hello
          method: get
```

## Key Configuration Areas

### Provider Settings

AWS-specific provider configuration in `serverless.yml`:

```yaml
provider:
  name: aws
  runtime: nodejs20.x           # Lambda runtime
  stage: ${opt:stage, 'dev'}    # Deployment stage
  region: ${opt:region, 'us-east-1'}
  memorySize: 512               # Default memory (MB)
  timeout: 30                     # Default timeout (seconds)
  environment:                  # Lambda env vars
    NODE_ENV: ${sls:stage}
  iam:                          # IAM role configuration
    role:
      statements:
        - Effect: Allow
          Action:
            - dynamodb:GetItem
          Resource: '*'
  vpc:                          # VPC configuration
    securityGroupIds:
      - sg-xxxxxxxxx
    subnetIds:
      - subnet-xxxxxxxxx
  logs:                         # CloudWatch logs
    lambda:
      logFormat: JSON
    restApi:
      accessLogging: true
      level: INFO
```

### Functions

Functions are the core unit of compute:

```yaml
functions:
  myFunction:
    handler: src/users.create     # file.function
    runtime: nodejs20.x           # Override provider runtime
    memorySize: 512               # Override provider memory
    timeout: 30                   # Override provider timeout
    environment:                  # Function-specific env vars
      TABLE_NAME: ${self:custom.tableName}
    events:                       # Event triggers (see Events section)
      - httpApi:
          path: /users
          method: post
    iam:                          # Per-function IAM role
      role:
        statements:
          - Effect: Allow
            Action:
              - s3:GetObject
            Resource: arn:aws:s3:::${self:custom.bucket}/*
```

### Events

Common event triggers:

**HTTP API (API Gateway v2):**
```yaml
events:
  - httpApi:
      path: /users/{id}
      method: get
      authorizer: jwtAuthorizer   # Reference to provider.httpApi.authorizers
```

**REST API (API Gateway v1):**
```yaml
events:
  - http:
      path: users/{id}
      method: get
      cors: true
      authorizer:
        name: main-authorizer
        resultTtlInSeconds: 0
      request:
        parameters:
          paths:
            id: true
```

**Schedule (CloudWatch Events):**
```yaml
events:
  - schedule:
      rate: rate(5 minutes)
      input:
        key: value
```

**S3:**
```yaml
events:
  - s3:
      bucket: my-bucket
      event: s3:ObjectCreated:*
      rules:
        - prefix: uploads/
        - suffix: .jpg
```

**SQS:**
```yaml
events:
  - sqs:
      arn: arn:aws:sqs:region:account:queue-name
      batchSize: 10
```

**SNS:**
```yaml
events:
  - sns:
      topicName: my-topic
      filterPolicy:
        status:
          - completed
```

**DynamoDB Streams:**
```yaml
events:
  - stream:
      type: dynamodb
      arn: arn:aws:dynamodb:region:account:table/table-name/stream/...
      filterPatterns:
        - eventName: [INSERT]
```

See [references/events.md](references/events.md) for complete event reference.

### Plugins

Plugins extend the framework:

```yaml
plugins:
  - serverless-offline        # Local development
  - serverless-esbuild         # Bundling with esbuild
  - serverless-compose         # Multi-service deployment

custom:
  esbuild:
    bundle: true
    minify: true
    sourcemap: true
```

## Variables

Serverless Framework supports multiple variable sources:

```yaml
# CLI options
stage: ${opt:stage, 'dev'}

# Environment variables
apiKey: ${env:API_KEY}

# Self-references
serviceName: ${self:service}

# File references
config: ${file(./config.yml)}

# AWS CloudFormation outputs
queueUrl: ${cf:another-service.QueueUrl}

# SSM Parameter Store
dbHost: ${ssm:/prod/db/host}

# Secrets Manager
apiSecret: ${secretsmanager:/prod/api/secret}
```

See [references/variables.md](references/variables.md) for complete variable reference.

## Packaging

Control what gets deployed:

```yaml
package:
  individually: true            # Package each function separately
  patterns:
    - src/**                    # Include
    - '!.git/**'                # Exclude
    - '!.env'
    - '!tests/**'
```

## CLI Commands

### Deployment

```bash
# Deploy entire service
serverless deploy

# Deploy to specific stage/region
serverless deploy --stage prod --region eu-west-1

# Deploy single function (faster)
serverless deploy function --function myFunction

# Package without deploying
serverless package
```

### Development

```bash
# Local development server
serverless dev

# Invoke function locally
serverless invoke local --function myFunction --path event.json

# Invoke deployed function
serverless invoke --function myFunction --data '{"key":"value"}'

# Tail logs
serverless logs --function myFunction --tail

# View metrics
serverless metrics --function myFunction
```

### Info & Management

```bash
# View service info
serverless info

# View CloudFormation template
serverless print

# Remove service
serverless remove

# Rollback deployment
serverless rollback --timestamp 1234567890

# List deployments
serverless deploy list
```

See [references/cli.md](references/cli.md) for complete CLI reference.

## Multi-Service Projects

### Serverless Compose

For monorepos with multiple services:

```yaml
# serverless-compose.yml
services:
  api:
    path: services/api

  workers:
    path: services/workers
    dependsOn: api
    params:
      apiUrl: ${api.apiGatewayRestApiUrl}

  frontend:
    path: services/frontend
    dependsOn:
      - api
      - workers
```

Key features:
- Deploy services in parallel or ordered by dependencies
- Share outputs between services via `${service.output}`
- Run commands across all services: `serverless deploy` at root
- Service-specific commands: `serverless api deploy`

## Serverless MCP Server

The Serverless MCP Server provides AI-assisted debugging and infrastructure management through the Model Context Protocol.

### Setup

Requires Serverless Framework v4.13.0+:

```bash
npm install -g serverless
```

**Cursor/Windsurf stdio configuration:**
```json
{
  "mcpServers": {
    "serverless": {
      "command": "serverless",
      "args": ["mcp"]
    }
  }
}
```

**SSE transport:**
```bash
serverless mcp --transport sse
```

### MCP Tools

The MCP server provides tools for infrastructure debugging:

**Cross-Cloud Tools:**
- `list-projects` - Discover serverless projects in workspace
- `list-resources` - List all resources in a service
- `service-summary` - Comprehensive service overview
- `deployment-history` - View deployment events
- `docs` - Access Serverless Framework documentation

**AWS Service Tools:**
- `aws-lambda-info` - Lambda configuration, metrics, logs
- `aws-iam-info` - IAM roles and policies
- `aws-sqs-info` - SQS queue details
- `aws-s3-info` - S3 bucket configuration
- `aws-rest-api-gateway-info` - REST API Gateway details
- `aws-http-api-gateway-info` - HTTP API Gateway details
- `aws-dynamodb-info` - DynamoDB table metrics
- `aws-logs-search` - CloudWatch Logs Insights queries
- `aws-logs-tail` - Recent CloudWatch logs
- `aws-cloudwatch-alarms` - Alarm history
- `aws-errors-info` - Error pattern analysis

See [references/mcp-tools.md](references/mcp-tools.md) for complete tool reference.

### AWS Credentials

The MCP server uses the standard AWS credential chain:
1. Environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
2. AWS SSO credentials
3. Shared credentials file (`~/.aws/credentials`)

**Always specify profile explicitly** when using MCP tools in multi-account environments.

**Minimum IAM permissions:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "lambda:List*", "lambda:Get*",
      "iam:List*", "iam:Get*",
      "sqs:List*", "sqs:Get*",
      "s3:List*", "s3:Get*",
      "apigateway:GET", "apigatewayv2:Get*",
      "dynamodb:List*", "dynamodb:Describe*",
      "cloudwatch:Get*", "cloudwatch:Describe*",
      "logs:FilterLogEvents", "logs:StartQuery", "logs:GetQueryResults"
    ],
    "Resource": "*"
  }]
}
```

## Best Practices

### Project Structure

```
src/
  functions/              # Lambda handlers
    users/
      create.ts
      list.ts
  lib/                    # Shared code
    db.ts
    logger.ts
  types/                  # TypeScript types
tests/                    # Test files
serverless.yml
package.json
```

### Configuration Tips

- Use stages (`dev`, `staging`, `prod`) with different AWS accounts
- Set `package.individually: true` for faster deployments
- Use `${file()}` to split large configurations
- Keep functions focused (single responsibility)
- Use environment variables for configuration, not code

### Security

- Never commit `.env` files or credentials
- Use IAM roles over access keys where possible
- Apply least-privilege IAM permissions
- Enable CloudTrail for audit logging
- Use VPC for sensitive workloads

### Performance

- Set appropriate memory/timeout values
- Use provisioned concurrency for latency-sensitive functions
- Enable X-Ray tracing for distributed debugging
- Package only required files
- Use Lambda layers for shared dependencies

## Resources

- [Serverless Framework Documentation](https://www.serverless.com/framework/docs)
- [Serverless Plugins Registry](https://www.serverless.com/plugins)
- [AWS Lambda Limits](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
- [Serverless Compose Guide](https://www.serverless.com/framework/docs/guides/compose)

## Reference Files

- [references/serverless-yml.md](references/serverless-yml.md) - Complete serverless.yml property reference
- [references/events.md](references/events.md) - All event trigger configurations
- [references/cli.md](references/cli.md) - CLI command reference
- [references/variables.md](references/variables.md) - Variable resolution reference
- [references/mcp-tools.md](references/mcp-tools.md) - MCP Server tools documentation
