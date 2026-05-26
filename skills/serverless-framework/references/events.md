# Events Reference

Complete reference for all AWS Lambda event triggers supported by Serverless Framework.

## HTTP API (API Gateway v2)

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - httpApi:
          path: /users/{id}
          method: get
          authorizer: jwtAuthorizer    # References provider.httpApi.authorizers
          cors: true                   # Uses provider.httpApi.cors settings
```

Properties:
- `path`: Route path with optional parameters (`{param}`)
- `method`: HTTP method (GET, POST, PUT, DELETE, PATCH, ANY)
- `authorizer`: Name of authorizer defined in `provider.httpApi.authorizers`
- `cors`: Enable CORS (boolean)

## REST API (API Gateway v1)

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - http:
          path: users/{id}
          method: get
          cors: true
          private: true                # Requires API key
          authorizer:
            name: authorizerFunc       # Function in same service
            arn: arn:aws:lambda:...    # External function
            resultTtlInSeconds: 0
            identitySource: method.request.header.Authorization
            identityValidationExpression: someRegex
            type: token                # or 'request'
          request:
            parameters:
              paths:
                id: true               # Required path param
              headers:
                headerName: true
                custom-header:
                  required: true
                  mappedValue: context.requestId
              querystrings:
                paramName: true
            schemas:
              application/json: ${file(schema.json)}
            template:
              application/json: '{ "httpMethod" : "$context.httpMethod" }'
            passThrough: NEVER
          response:
            transferMode: STREAM        # or 'BUFFERED'
```

## WebSocket API

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - websocket:
          route: $connect
          routeResponseSelectionExpression: $default
          authorizer:
            name: auth
            arn: arn:aws:lambda:...
            identitySource:
              - 'route.request.header.Auth'
              - 'route.request.querystring.Auth'
```

## S3

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - s3:
          bucket: photos
          event: s3:ObjectCreated:*
          rules:
            - prefix: uploads/
            - suffix: .jpg
          existing: true              # Use existing bucket
          forceDeploy: true           # Force trigger deployment
```

## Schedule (CloudWatch Events)

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - schedule:
          name: my-scheduled-event
          description: Event description
          rate: rate(10 minutes)      # or cron(0 12 * * ? *)
          enabled: true
          input:
            key1: value1
          inputPath: '$.stageVariables'
          inputTransformer:
            inputPathsMap:
              eventTime: '$.time'
            inputTemplate: '{"time": <eventTime>}'
```

## SNS

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - sns:
          topicName: aggregate
          displayName: Data aggregation pipeline
          filterPolicy:
            pet:
              - dog
              - cat
          filterPolicyScope: MessageAttributes
          redrivePolicy:
            deadLetterTargetArn: arn:aws:sqs:...
            deadLetterTargetRef: myDLQ
            deadLetterTargetImport:
              arn: MyShared-DLQArn
              url: MyShared-DLQUrl
```

## SQS

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - sqs:
          arn: arn:aws:sqs:region:XXXXXX:myQueue
          batchSize: 10               # 1-10000
          maximumBatchingWindow: 10   # 0-300 seconds
          enabled: true
          functionResponseType: ReportBatchItemFailures
          filterPatterns:
            - a: [1, 2]
```

## DynamoDB Streams

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - stream:
          type: dynamodb
          arn: arn:aws:dynamodb:region:XXXXXX:table/table-name/stream/...
          batchSize: 100
          maximumRecordAgeInSeconds: 120
          startingPosition: LATEST    # or TRIM_HORIZON, AT_TIMESTAMP
          enabled: true
          functionResponseType: ReportBatchItemFailures
          filterPatterns:
            - partitionKey: [1]
```

## Kinesis Streams

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - stream:
          type: kinesis
          arn: arn:aws:kinesis:region:XXXXXX:stream/foo
          batchSize: 100
          startingPosition: LATEST
          enabled: true
          filterPatterns:
            - partitionKey: [1]
```

## MSK (Managed Kafka)

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - msk:
          arn: arn:aws:kafka:us-east-1:111111111:cluster/ClusterName/...
          topic: kafkaTopic
          batchSize: 100
          maximumBatchingWindow: 30
          startingPosition: LATEST
          saslScram512: arn:aws:secretsmanager:...
          consumerGroupId: MyConsumerGroupId
```

## Kafka (Self-Managed)

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - kafka:
          accessConfigurations:
            saslScram256Auth:
              saslScram256: arn:aws:secretsmanager:...
          bootstrapServers:
            - abc3.xyz.com:9092
            - abc2.xyz.com:9092
          topic: MySelfManagedKafkaTopic
          batchSize: 100
          startingPosition: LATEST
```

## RabbitMQ

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - rabbitmq:
          arn: arn:aws:mq:us-east-1:0000:broker:ExampleMQBroker:b-xxx-xxx
          queue: queue-name
          virtualHost: virtual-host
          basicAuthArn: arn:aws:secretsmanager:...
          batchSize: 100
          maximumBatchingWindow: 30
```

## ActiveMQ

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - activemq:
          arn: arn:aws:mq:us-east-1:0000:broker:ExampleMQBroker:b-xxx-xxx
          queue: queue-name
          basicAuthArn: arn:aws:secretsmanager:...
          batchSize: 100
          maximumBatchingWindow: 30
```

## CloudWatch Event

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - cloudwatchEvent:
          event:
            source:
              - aws.codebuild
            detail:
              state:
                - SUCCEEDED
                - FAILED
          input:
            key1: value1
          inputPath: '$.detail'
          inputTransformer:
            inputPathsMap:
              state: '$.detail.state'
            inputTemplate: '{"state": <state>}'
```

## CloudWatch Log

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - cloudwatchLog:
          logGroup: /aws/lambda/my-function
          filterPattern: '{ $.errorType = "*" }'
```

## EventBridge

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - eventBridge:
          eventBus: arn:aws:events:us-east-1:12345:event-bus/custom
          schedule: rate(10 minutes)
          pattern:
            source:
              - aws.codecommit
            detail:
              event:
                - referenceCreated
                - referenceUpdated
          input:
            key1: value1
          inputPath: '$.detail'
          inputTransformer:
            inputPathsMap:
              branch: '$.detail.referenceName'
            inputTemplate: '{"branch": <branch>}'
```

## Cognito User Pool

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - cognitoUserPool:
          pool: MyUserPool
          trigger: PreSignUp
          existing: true
```

## IoT

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - iot:
          name: myIoTEvent
          description: An IoT event
          sql: "SELECT * FROM 'some_topic'"
          sqlVersion: beta
          enabled: true
```

## IoT Fleet Provisioning

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - iotFleetProvisioning:
          templateBody: '{"Parameters"...}'
          provisioningRoleArn: arn:aws:iam::12345:role/ProvisioningRole
          enabled: true
```

## Alexa Skill

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - alexaSkill:
          appId: amzn1.ask.skill.xx-xx-xx-xx
          enabled: true
```

## Alexa Smart Home

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - alexaSmartHome:
          appId: amzn1.ask.skill.xx-xx-xx-xx
          enabled: true
```

## ALB (Application Load Balancer)

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - alb:
          listenerArn: arn:aws:elasticloadbalancing:us-east-1:12345:listener/app/...
          priority: 1
          conditions:
            path: /hello
            host: example.com
            method: GET
            header:
              name: x-api-key
              values:
                - abc123
            query:
              bar:
                - bar1
          healthCheck:
            path: /health
            intervalSeconds: 5
            timeoutSeconds: 4
            healthyThresholdCount: 2
            unhealthyThresholdCount: 3
            matcher:
              httpCode: 200
```

## CloudFront (Lambda@Edge)

```yaml
functions:
  hello:
    handler: handler.hello
    events:
      - cloudfront:
          eventType: viewer-request
          origin: s3://bucketname.s3.amazonaws.com/files
          pathPattern: /files/*
          cachePolicy:
            name: myCachePolicy
```

## Multiple Events

A single function can have multiple event triggers:

```yaml
functions:
  processData:
    handler: handler.process
    events:
      - httpApi:
          path: /process
          method: post
      - schedule:
          rate: rate(1 hour)
      - sqs:
          arn: arn:aws:sqs:region:account:queue-name
```
