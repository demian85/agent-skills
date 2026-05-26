# Variables Reference

Serverless Framework provides a powerful variable system to make configurations dynamic and reusable.

## Variable Syntax

```yaml
${source:key, fallback}
```

- `source`: The variable source
- `key`: The key or path to retrieve
- `fallback`: Optional fallback value if resolution fails

## Core Variables

### `sls` - Serverless Built-ins

```yaml
${sls:stage}          # Current stage (dev, prod, etc.)
${sls:service}        # Service name
${sls:instanceId}       # Unique instance ID
```

### `self` - Self-Reference

Reference other parts of the same serverless.yml:

```yaml
service: my-service
provider:
  name: aws
  environment:
    SERVICE_NAME: ${self:service}
    REGION: ${self:provider.region}
```

### `opt` - CLI Options

Reference CLI options passed via command line:

```yaml
stage: ${opt:stage, 'dev'}           # Default to 'dev'
region: ${opt:region, 'us-east-1'}  # Default to 'us-east-1'
```

Usage:
```bash
serverless deploy --stage prod --region eu-west-1
```

### `env` - Environment Variables

```yaml
apiKey: ${env:API_KEY}
secretKey: ${env:SECRET_KEY, 'default-value'}
```

### `file` - File References

Load values from external files:

```yaml
# Load entire file
config: ${file(./config.yml)}

# Load specific property
dbHost: ${file(./config.yml):database.host}

# Load JSON
schema: ${file(./schema.json)}
```

Supported formats: YAML, JSON, JS, TS

### `param` - Parameters

Reference stage parameters:

```yaml
# serverless.yml
provider:
  environment:
    STRIPE_KEY: ${param:stripe_api_key}

# serverless-compose.yml or stage config
stages:
  prod:
    params:
      stripe_api_key: ${env:PROD_STRIPE_KEY}
  dev:
    params:
      stripe_api_key: ${env:DEV_STRIPE_KEY}
```

## AWS Variables

### `cf` - CloudFormation Outputs

Reference outputs from CloudFormation stacks:

```yaml
# Reference output from another service
queueUrl: ${cf:another-service.QueueUrl}
queueUrl: ${cf:another-service-${sls:stage}.QueueUrl}
```

### `aws` - AWS Account Info

```yaml
accountId: ${aws:accountId}      # AWS account ID
region: ${aws:region}              # Current region
```

### `ssm` - Systems Manager Parameter Store

```yaml
# By path
dbHost: ${ssm:/prod/db/host}

# With specific region
dbHost: ${ssm(us-west-2):/prod/db/host}

# SecureString (decrypted automatically)
dbPassword: ${ssm:/prod/db/password}
```

### `secretsmanager` - Secrets Manager

```yaml
# Get secret value
apiSecret: ${secretsmanager:/prod/api/secret}

# Get specific key from JSON secret
dbPassword: ${secretsmanager:/prod/db/credentials~password}

# With region
apiSecret: ${secretsmanager(us-west-2):/prod/api/secret}
```

### `s3` - S3 Bucket Objects

```yaml
config: ${s3:my-bucket/config.yml}
```

## Custom Variable Resolvers

### JavaScript/TypeScript Variables

Create a `variables.js` or `variables.ts` file:

```javascript
// variables.js
module.exports = {
  resolve: async ({ options, resolveConfigurationProperty }) => {
    // Custom resolution logic
    return 'resolved-value'
  }
}
```

Use in serverless.yml:
```yaml
customValue: ${file(./variables.js)}
```

### Git Variables

Available via `serverless-plugin-git-variables`:

```yaml
sha: ${git:sha1}              # Current commit SHA
branch: ${git:branch}          # Current branch
tag: ${git:describeLight}      # Current tag
```

### Doppler Variables

Available via `serverless-doppler-plugin`:

```yaml
apiKey: ${doppler:API_KEY}
```

### Terraform Variables

Available via plugins:

```yaml
output: ${terraform:my-output}
```

### HashiCorp Vault

Available via plugins:

```yaml
secret: ${vault:/path/to/secret}
```

## Variable Modifiers

### Fallback Values

```yaml
# Fallback if variable is not found
stage: ${opt:stage, 'dev'}
region: ${opt:region, 'us-east-1'}

# Fallback with another variable
apiKey: ${env:API_KEY, ${ssm:/prod/api/key}}
```

### Property Access

```yaml
# Access nested properties
${file(./config.yml):database.host}
${self:custom.config.database.host}
```

### Spread Operator

```yaml
# Merge objects
custom:
  baseConfig: ${file(./base-config.yml)}
  envConfig: ${file(./${sls:stage}-config.yml)}
  merged: ${self:custom.baseConfig, self:custom.envConfig}
```

## Variable Sources Priority

When multiple sources provide the same parameter, the priority is:

1. CLI options (`--param` flag)
2. Stage parameters in `serverless.yml`
3. Environment variables
4. Parameters from `serverless-compose.yml`
5. Default values

## Advanced Patterns

### Conditional Configuration

```yaml
provider:
  environment:
    DEBUG: ${self:custom.debug.${sls:stage}}

custom:
  debug:
    dev: 'true'
    prod: 'false'
```

### Dynamic Imports

```yaml
functions: ${file(./functions.yml)}
resources: ${file(./resources.yml)}
```

### Cross-Service References

```yaml
# In service-b/serverless.yml
environment:
  API_URL: ${cf:service-a-${sls:stage}.ServiceUrl}
```

### Array/Map Operations

```yaml
# Using JavaScript to transform
custom:
  regions:
    - us-east-1
    - us-west-2
    - eu-west-1
  regionMap: ${file(./regions-map.js)}
```

## Common Pitfalls

1. **Circular References**: Avoid referencing variables that reference each other
2. **Missing Sources**: Always provide fallbacks for optional variables
3. **Region Mismatch**: Ensure SSM/Secrets Manager variables use the correct region
4. **Permissions**: Ensure Lambda execution role has permissions to access SSM/Secrets Manager
5. **Cold Start**: Frequent SSM/Secrets Manager lookups can increase cold start time; use environment variables or cache
