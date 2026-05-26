# Serverless.yml Reference

Complete reference for all `serverless.yml` properties when using the AWS provider.

## Table of Contents

- [Root Properties](#root-properties)
- [Provider](#provider)
- [Package](#package)
- [Functions](#functions)
- [Resources](#resources)
- [Plugins](#plugins)
- [Custom](#custom)

## Root Properties

```yaml
org: my-org                    # Serverless Framework org (optional)
app: my-app                    # App name for Dashboard features (optional)
service: my-service            # Service name (required)
```

### Stages

Stage-specific configuration:

```yaml
stages:
  prod:
    observability: true
    params:
      stripe_api_key: ${env:PROD_STRIPE_API_KEY}
  default:
    observability: false
    params:
      stripe_api_key: ${env:DEV_STRIPE_API_KEY}
```

### Parameters

```yaml
params:
  default:
    domain: ${sls:stage}.myapi.com
  prod:
    domain: myapi.com
```

## Provider

### General Settings

```yaml
provider:
  name: aws
  stage: ${opt:stage, 'dev'}
  region: ${opt:region, 'us-east-1'}
  profile: production
  stackName: custom-stack-name
  tags:
    foo: bar
  stackTags:
    key: value
  deploymentMethod: direct    # or 'changesets'
  notificationArns:
    - 'arn:aws:sns:us-east-1:XXXXXX:mytopic'
  stackParameters:
    - ParameterKey: 'Keyname'
      ParameterValue: 'Value'
  disableRollback: true
  resolver: aws-account-1
  tracing:
    apiGateway: true
    lambda: true                # or 'Active' or 'PassThrough'
```

### Lambda Settings

```yaml
provider:
  runtime: nodejs20.x
  runtimeManagement: auto      # or 'onFunctionUpdate'
  memorySize: 512              # Default: 1024MB
  timeout: 10                  # Default: 6 seconds
  environment:
    APP_ENV_VARIABLE: FOOBAR
  logRetentionInDays: 14
  logDataProtectionPolicy:
    Name: data-protection-policy
  kmsKeyArn: arn:aws:kms:...
  versionFunctions: false
  architecture: x86_64         # or 'arm64'
```

### Deployment Bucket

```yaml
provider:
  deploymentPrefix: serverless
  deploymentBucket:
    name: com.serverless.${self:provider.region}.deploys
    maxPreviousDeploymentArtifacts: 10
    blockPublicAccess: true
    skipPolicySetup: true
    versioning: true
    serverSideEncryption: AES256
    sseKMSKeyId: arn:aws:kms:...
  enableLegacyDeploymentBucket: false
```

### API Gateway v2 (HTTP API)

```yaml
provider:
  httpApi:
    id: xxxx                    # Attach existing API
    name: dev-my-service
    payload: '2.0'
    disableDefaultEndpoint: true
    metrics: true
    cors: true                  # or detailed config
    authorizers:
      jwtAuth:
        identitySource: $request.header.Authorization
        issuerUrl: https://cognito-idp...
        audience:
          - xxxx
      lambdaAuth:
        type: request
        functionName: authorizerFunc
        resultTtlInSeconds: 300
        enableSimpleResponses: true
```

### API Gateway v1 (REST API)

```yaml
provider:
  apiName: custom-api-name
  endpointType: REGIONAL         # or 'EDGE'
  apiGateway:
    restApiId: xxxx
    restApiRootResourceId: xxxx
    restApiResources:
      '/users': xxxx
    disableDefaultEndpoint: true
    apiKeySourceType: HEADER
    apiKeys:
      - name: myKey
        value: myKeyValue
    minimumCompressionSize: 1024
    metrics: false
    shouldStartNameWithService: false
    resourcePolicy:
      - Effect: Allow
        Principal: '*'
        Action: execute-api:Invoke
        Resource:
          - execute-api:/*/*/*
    usagePlan:
      quota:
        limit: 5000
        offset: 2
        period: MONTH
      throttle:
        burstLimit: 200
        rateLimit: 100
```

### ALB

```yaml
provider:
  alb:
    targetGroupPrefix: xxxx
    authorizers:
      cognitoAuth:
        type: cognito
        userPoolArn: arn:aws:cognito-idp:...
        userPoolClientId: xxx
        userPoolDomain: your-domain
        onUnauthenticatedRequest: deny
      oidcAuth:
        type: oidc
        authorizationEndpoint: https://...
        clientId: xxx
        issuer: https://...
```

### Docker/ECR

```yaml
provider:
  ecr:
    scanOnPush: true
    images:
      baseimage:
        uri: 000000000000.dkr.ecr.us-east-1.amazonaws.com/test-image@sha256:...
      custombuild:
        path: ./image/
        file: Dockerfile.dev
        buildArgs:
          STAGE: ${sls:stage}
```

### CloudFront

```yaml
provider:
  cloudFront:
    cachePolicies:
      myPolicy:
        DefaultTTL: 60
        MinTTL: 30
        MaxTTL: 3600
        ParametersInCacheKeyAndForwardedToOrigin:
          CookiesConfig:
            CookieBehavior: whitelist
            Cookies:
              - my-public-cookie
          HeadersConfig:
            HeaderBehavior: whitelist
            Headers:
              - authorization
```

### IAM

```yaml
provider:
  iam:
    role: arn:aws:iam::XXXXXX:role/role    # Existing role
    # OR configure created role:
    role:
      statements:
        - Effect: Allow
          Action:
            - s3:ListBucket
          Resource: arn:aws:s3:::bucket
      name: custom-role-name
      path: /custom-path/
      managedPolicies:
        - arn:aws:iam::aws:policy/ReadOnlyAccess
      permissionsBoundary: arn:aws:iam::XXXXXX:policy/boundary
      tags:
        key: value
    deploymentRole: arn:aws:iam::XXXXXX:role/cloudformation-role
  stackPolicy:
    - Effect: Allow
      Principal: '*'
      Action: 'Update:*'
      Resource: '*'
```

### VPC

```yaml
provider:
  vpc:
    ipv6AllowedForDualStack: true
    securityGroupIds:
      - sg-xxxxxxxxx
    subnetIds:
      - subnet-xxxxxxxxx
```

### Logs

```yaml
provider:
  logs:
    lambda:
      logFormat: JSON             # or 'Text'
      applicationLogLevel: ERROR
      systemLogLevel: INFO
      logGroup: /aws/lambda/global
    httpApi:
      format: '{ "requestId":"$context.requestId" }'
    restApi:
      accessLogging: true
      executionLogging: true
      level: INFO
      fullExecutionData: true
      role: arn:aws:iam::123456:role
      roleManagedExternally: false
    websocket:
      accessLogging: true
      executionLogging: true
      level: INFO
      fullExecutionData: true
    frameworkLambda: false
```

### S3 Buckets

```yaml
provider:
  s3:
    bucketOne:
      name: my-custom-bucket-name
      versioningConfiguration:
        Status: Enabled
```

## Package

```yaml
package:
  patterns:
    - src/**
    - handler.js
    - '!.git/**'
    - '!.travis.yml'
  individually: true
  artifact: path/to/artifact.zip
  excludeDevDependencies: false
```

## Functions

### Basic Function

```yaml
functions:
  hello:
    handler: users.create
    runtime: nodejs20.x
    memorySize: 512
    timeout: 10
    environment:
      APP_ENV_VARIABLE: FOOBAR
    name: ${sls:stage}-lambdaName
    description: My function
    architecture: x86_64
```

### Advanced Configuration

```yaml
functions:
  hello:
    handler: users.create
    ephemeralStorageSize: 512     # MB (default: 512)
    reservedConcurrency: 5        # Max concurrent executions
    provisionedConcurrency:       # Min concurrent executions
      alias: active
      executions: 3
    role: arn:aws:iam::XXXXXX:role/role
    iam:
      inheritStatements: true
      role:
        statements:
          - Effect: Allow
            Action:
              - dynamodb:GetItem
            Resource: arn:aws:dynamodb:*:*:table/Users
        managedPolicies:
          - arn:aws:iam::aws:policy/ReadOnlyAccess
    onError: arn:aws:sns:us-east-1:XXXXXX:sns-topic
    kmsKeyArn: arn:aws:kms:...
    snapStart: true               # Java only
    disableLogs: false
    logRetentionInDays: 14
    logs:
      logFormat: JSON
      applicationLogLevel: ERROR
      systemLogLevel: INFO
    layers:
      - arn:aws:lambda:region:XXXXXX:layer:LayerName:1
    tracing: Active
    condition: SomeCondition
    dependsOn:
      - MyThing
    destinations:
      onSuccess: functionName
      onFailure: arn:xxx:target
    fileSystemConfig:
      arn: arn:aws:elasticfilesystem:us-east-1:11111111:access-point/fsap-...
      localMountPath: /mnt/example
    maximumRetryAttempts: 1       # 0-2
    maximumEventAge: 7200         # 60-21600 seconds
```

### URL Configuration

```yaml
functions:
  hello:
    url:
      authorizer: 'aws_iam'
      cors:
        allowedOrigins:
          - '*'
        allowedHeaders:
          - Authorization
        allowedMethods:
          - GET
        allowCredentials: true
        exposedResponseHeaders:
          - SomeHeader
        maxAge: 3600
```

## Resources

Add raw CloudFormation resources:

```yaml
resources:
  Resources:
    MyDynamoDBTable:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: ${self:service}-table-${sls:stage}
        AttributeDefinitions:
          - AttributeName: id
            AttributeType: S
        KeySchema:
          - AttributeName: id
            KeyType: HASH
        BillingMode: PAY_PER_REQUEST
  Outputs:
    TableName:
      Value: !Ref MyDynamoDBTable
      Export:
        Name: ${self:service}-table-name-${sls:stage}
```

## Plugins

```yaml
plugins:
  - serverless-offline
  - serverless-esbuild
  - ./local-plugin

custom:
  esbuild:
    bundle: true
    minify: true
    sourcemap: true
    target: node20
```

## Layers

```yaml
layers:
  myLayer:
    path: layer
    name: ${sls:stage}-myLayer
    description: My layer
    compatibleRuntimes:
      - nodejs20.x
    retain: false
```
