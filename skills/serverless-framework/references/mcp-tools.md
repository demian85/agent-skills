# MCP Server Tools Reference

The Serverless MCP Server provides AI assistants with tools to debug and manage AWS serverless infrastructure.

## Setup

Requires Serverless Framework v4.13.0+:

```bash
npm install -g serverless
```

## Configuration

### Cursor (stdio)

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

### Cursor (SSE)

```bash
# Start server
serverless mcp --transport sse
```

```json
{
  "mcpServers": {
    "serverless": {
      "url": "http://localhost:3001/sse"
    }
  }
}
```

### Windsurf (stdio)

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

### Windsurf (SSE)

```bash
serverless mcp --transport sse
```

```json
{
  "mcpServers": {
    "serverless": {
      "url": "http://localhost:3001/sse"
    }
  }
}
```

## Cross-Cloud Tools

### `list-projects`

Discovers serverless projects in the workspace.

**Parameters:**
- `workspaceRoots` (string[]): Root directories to search
- `userConfirmed` (boolean): User confirmation to search

**Returns:**
- List of projects with type, path, and configuration

### `list-resources`

Lists all resources defined in infrastructure configuration.

**Parameters:**
- `serviceName` (string): Service name (format: "serviceName-stageName" for Serverless Framework)
- `serviceType` (string): "serverless-framework", "terraform", or "cloudformation"
- `region` (string, optional): AWS region
- `profile` (string, optional): AWS profile

**Returns:**
- Lambda functions, API Gateway endpoints, DynamoDB tables, S3 buckets, IAM roles, event sources
- Resource types, identifiers, and configuration details

### `service-summary`

Comprehensive overview of an entire service.

**Parameters:**
- `serviceType` (string): Cloud provider ('aws', 'gcp', 'azure')
- `serviceWideAnalysis` (boolean): Analyze ALL resources automatically
- `serviceName` (string): Service name (required if serviceWideAnalysis is true)
- `cloudProvider` (string): "aws" (required if serviceWideAnalysis is true)
- `resources` (array, optional): Specific resources to analyze
- `startTime` (string, optional): Start time for metrics (ISO date)
- `endTime` (string, optional): End time for metrics
- `period` (number, optional): Metric period in seconds (default: 3600)
- `region` (string, optional): AWS region
- `profile` (string, optional): AWS profile

**Returns:**
- Consolidated view of all resources
- Configuration details, metrics, and logs

### `deployment-history`

Retrieves deployment history.

**Parameters:**
- `serviceName` (string): Service name
- `serviceType` (string): "serverless-framework" or "cloudformation"
- `endDate` (string, optional): End date (ISO format)
- `region` (string, optional): AWS region
- `profile` (string, optional): AWS profile

**Returns:**
- Chronological history of stack events
- Resource creation, updates, and deletions

### `docs`

Accesses up-to-date Serverless Framework documentation.

**Parameters:**
- `product` (string): "sf" (Serverless Framework) or "scf" (Serverless Container Framework)
- `paths` (string[], optional): Document paths to retrieve

**Returns:**
- Documentation content or directory listing
- Markdown content with examples

## AWS Service Tools

### `aws-lambda-info`

Diagnoses Lambda functions.

**Parameters:**
- `functionNames` (string[]): Function names or ARNs
- `startTime` (string, optional): Start time for metrics/logs
- `endTime` (string, optional): End time
- `period` (number, optional): Metric period (default: 3600)
- `region` (string, optional): AWS region
- `profile` (string, optional): AWS profile

**Returns:**
- Configuration details
- Performance metrics (invocations, duration, errors, throttles)
- Error logs with stack traces
- Event source mappings
- Resource-based policies

### `aws-iam-info`

Retrieves IAM role details.

**Parameters:**
- `roleNames` (string[]): IAM role names
- `region` (string, optional): AWS region
- `profile` (string, optional): AWS profile

**Returns:**
- Trust policies
- Attached managed policies
- Inline policies with full JSON
- Permission boundaries
- Last used information

### `aws-sqs-info`

Analyzes SQS queues.

**Parameters:**
- `queueNames` (string[]): Queue names or URLs
- `startTime` (string, optional): Start time
- `endTime` (string, optional): End time
- `period` (number, optional): Metric period (default: 3600)
- `region` (string, optional): AWS region
- `profile` (string, optional): AWS profile

**Returns:**
- Queue attributes
- CloudWatch metrics (messages sent/received/deleted)
- Dead-letter queue configuration
- Visibility timeout and retention settings

### `aws-s3-info`

Retrieves S3 bucket details.

**Parameters:**
- `bucketNames` (string[]): Bucket names
- `startTime` (string, optional): Start time
- `endTime` (string, optional): End time
- `period` (number, optional): Metric period (default: 3600)
- `region` (string, optional): AWS region
- `profile` (string, optional): AWS profile

**Returns:**
- Configuration and ACLs
- Bucket policies and CORS
- Versioning status
- Encryption settings
- Lifecycle rules
- CloudWatch metrics

### `aws-rest-api-gateway-info`

Analyzes REST API Gateways (v1).

**Parameters:**
- `apiIds` (string[]): REST API IDs
- `startTime` (string, optional): Start time
- `endTime` (string, optional): End time
- `period` (number, optional): Metric period (default: 3600)
- `region` (string, optional): AWS region
- `profile` (string, optional): AWS profile

**Returns:**
- API configuration
- Stages, resources, methods
- Integration configurations
- Deployments and API keys
- Usage plans and VPC links

### `aws-http-api-gateway-info`

Analyzes HTTP API Gateways (v2).

**Parameters:**
- `apiIds` (string[]): HTTP API IDs
- `startTime` (string, optional): Start time
- `endTime` (string, optional): End time
- `period` (number, optional): Metric period (default: 3600)
- `region` (string, optional): AWS region
- `profile` (string, optional): AWS profile

**Returns:**
- Routes and integrations
- Stages and authorizers
- Metrics and error rates
- Logging configuration

### `aws-dynamodb-info`

Analyzes DynamoDB tables.

**Parameters:**
- `tableNames` (string[]): Table names
- `startTime` (string, optional): Start time
- `endTime` (string, optional): End time
- `period` (number, optional): Metric period (default: 3600)
- `region` (string, optional): AWS region
- `profile` (string, optional): AWS profile

**Returns:**
- Table configuration
- Provisioned capacity
- Global/local secondary indexes
- Stream settings
- Metrics (read/write capacity, latency, throttling)

## Logging Tools

### `aws-logs-search`

Searches CloudWatch logs using Logs Insights.

**Parameters:**
- `logGroupIdentifiers` (string[]): Log group names/ARNs
- `searchTerms` (string[], optional): Search terms (OR logic)
- `startTime` (string, optional): Start time (default: 3 hours ago)
- `endTime` (string, optional): End time (default: now)
- `limit` (number, optional): Max events per group (default: 100)
- `region` (string, optional): AWS region
- `profile` (string, optional): AWS profile

**Returns:**
- Filtered log events organized by log group

**Cost Note:** CloudWatch Logs Insights charges ~$0.005 per GB scanned. Use specific time ranges.

### `aws-logs-tail`

Retrieves recent CloudWatch logs.

**Parameters:**
- `logGroupIdentifiers` (string[]): Log group names/ARNs
- `filterPattern` (string, optional): CloudWatch filter pattern
- `startTime` (string, optional): Start time (default: 15 minutes ago)
- `endTime` (string, optional): End time (default: now)
- `limit` (number, optional): Max events per group (default: 100)
- `region` (string, optional): AWS region
- `profile` (string, optional): AWS profile

**Returns:**
- Recent log events

**Filter Pattern Examples:**
- `"ERROR"` - Contains ERROR
- `"ERROR TIMEOUT"` - Contains both
- `?ERROR ?TIMEOUT` - Contains either
- `{ $.errorCode = "NotFound" }` - JSON field match
- `{ $.responseTime > 5000 }` - Numeric comparison

### `aws-cloudwatch-alarms`

Gets CloudWatch alarm information.

**Parameters:**
- `alarmNames` (string[], optional): Specific alarm names
- `alarmNamePrefix` (string, optional): Name prefix filter
- `alarmState` (string, optional): "OK", "ALARM", "INSUFFICIENT_DATA", or "all"
- `startDate` (string, optional): Start date for history
- `endDate` (string, optional): End date for history
- `region` (string, optional): AWS region
- `profile` (string, optional): AWS profile

**Returns:**
- Alarm details and current state
- State change history

### `aws-errors-info`

Analyzes and groups error patterns.

**Parameters:**
- `logGroupIdentifiers` (string[], optional): Log groups to analyze
- `serviceWideAnalysis` (boolean): Analyze entire service
- `serviceName` (string, optional): Required if serviceWideAnalysis
- `serviceType` (string, optional): "serverless-framework" or "cloudformation"
- `startTime` (string, optional): Start time
- `endTime` (string, optional): End time
- `maxResults` (number, optional): Max error groups (default: 100)
- `confirmationToken` (string, optional): Required for >3 hour queries
- `region` (string, optional): AWS region
- `profile` (string, optional): AWS profile

**Returns:**
- Grouped error patterns with similarity matching
- Frequency statistics
- Example log entries per group
- Timeline of occurrences

**Cost Warning:** Queries >3 hours require user confirmation due to CloudWatch Logs Insights costs.

## AWS Credentials

The MCP server uses the standard AWS credential chain:
1. Environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
2. AWS SSO credentials
3. Shared credentials file (`~/.aws/credentials`)

### Important Notes

- **Always specify profile** in multi-account environments
- **Always provide region** if not in service configuration
- For Serverless Framework, check `provider.region` in serverless.yml
- SSO credentials expire after 8-12 hours; refresh with `aws sso login --profile <name>`

### Minimum IAM Permissions

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

## Cost Considerations

**Free Operations:**
- Listing resources
- Getting configurations
- Retrieving CloudWatch metrics
- Describing alarms
- Retrieving CloudWatch logs (FilterLogEvents)

**Paid Operations:**
- CloudWatch Logs Insights queries (`aws-logs-search`): ~$0.005 per GB scanned
- First 5 GB per month included in AWS Free Tier
- Use specific time ranges to minimize costs

## Troubleshooting

1. **Credentials Issues:**
   - Verify: `aws sts get-caller-identity`
   - Refresh SSO: `aws sso login --profile <name>`
   - Check IAM permissions

2. **Region Issues:**
   - Explicitly specify region in tool parameters
   - Check `provider.region` in serverless.yml

3. **Resource Not Found:**
   - Verify service name format: "serviceName-stageName"
   - Check if stack exists in target region
   - Confirm AWS profile/account
