# Cloud challenge - Stage 4
For this stage, the serverless framework has been applied.

## Background
Develop, deploy, troubleshoot and secure your serverless applications with radically less overhead and cost by using the Serverless Framework.
The Serverless Framework consists of an open source CLI and a hosted dashboard.
Together, they provide you with full serverless application lifecycle management.

When we are working with the serverless framework, we can create a template using different file formats. For this documentation and examples, the YAML format was used.

## Templates used for solution approach
The proposed solution here uses two lambda functions, around which the serverless templates will be centered.

### SQS - Lambda - SES
Important notes:

- `service:` is like a project. It's where you define your functions, events, resources and so on.
- `provider:` allows to configure the cloud provider (AWS, Azure, Google, etc.). Also, runtime, stage and region
- `provider.deploymentBucket:` allows to configure the s3 bucket used to load some required files for run the cloudformation stack.
This parameter is optional, but is important if we want to use the aws cloudformation console for delete the stack. 
- `iam:` Overwrite the default IAM role which is used for all functions.
- `package:` You can use the package and patterns configuration for more control over the packaging process.
- `functions:` as many AWS lambda functions as you want.
- `resources:` is similar to Resources node used in the AWS CloudFormation templates.

```yml
    service: challenge4-sqs-email
    
    provider:
      name: aws
      runtime: go1.x
      stage: dev
      region: us-east-2
      lambdaHashingVersion: 20201221
      deploymentBucket:
        name: 'serverless-deployment-bucket-wp'
      iam:
        role:
          statements:
            - Effect: Allow
              Action:
                - sqs:ReceiveMessage
                - ses:SendEmail
                - ses:SendRawEmail
                - ses:SendTemplatedEmail
                - ses:SendBulkTemplatedEmail
              Resource:
                - '*'
    
    package:
      patterns:
        - '!./**'
        - ./bin/**
    
    functions:
      sender:
        handler: bin/application
        name: 'serverless-sqs-lambda-ses'
        events:
          - sqs:
              arn:
                Fn::GetAtt:
                  - receiverQueue
                  - Arn
    resources:
      Resources:
        receiverQueue:
          Type: AWS::SQS::Queue
          Properties:
            QueueName: receiverQueue
```

### API Gateway - Lambda - DynamoDB
Important notes:

- `custom:` this section is used for parametrize the serverless template.
- `functions.application.environment:` here we can add the environment variables for AWS lambda functions 
- `functions.application.events:` here we can add the triggers of lambda function. In this case, the API Gateway.

```yml
    service: challenge4
    
    custom:
      userTableName: User
      attemptsTableName: LoginAttempt
      sqsUrl: 'https://sqs.us-east-1.amazonaws.com/379816580415/receiverQueue'
    
    provider:
      name: aws
      runtime: go1.x
      stage: dev
      region: us-east-2
      lambdaHashingVersion: 20201221
      deploymentBucket:
        name: 'serverless-deployment-bucket-wp'
    
    package:
      patterns:
        - '!./**'
        - ./bin/**
    
    functions:
      application:
        handler: bin/application
        role: 'arn:aws:iam::379816580415:role/lambda-role'
        name: 'serverless-lambda'
        environment:
          USER_TABLE_NAME: ${self:custom.userTableName}
          ATTEMPTS_TABLE_NAME: ${self:custom.attemptsTableName}
          QUEUE_URL: ${self:custom.sqsUrl}
        events:
          - http: ANY /
          - http: 'ANY /{proxy+}'
    
    resources:
      Resources:
        usersTable:
          Type: AWS::DynamoDB::Table
          Properties:
            TableName: ${self:custom.userTableName}
            AttributeDefinitions:
              - AttributeName: email
                AttributeType: S
            KeySchema:
              - AttributeName: email
                KeyType: HASH
            ProvisionedThroughput:
              ReadCapacityUnits: 1
              WriteCapacityUnits: 1
        attemptsTable:
          Type: AWS::DynamoDB::Table
          Properties:
            TableName: ${self:custom.attemptsTableName}
            AttributeDefinitions:
              - AttributeName: email
                AttributeType: S
              - AttributeName: time
                AttributeType: S
            KeySchema:
              - AttributeName: email
                KeyType: HASH
              - AttributeName: time
                KeyType: RANGE
            LocalSecondaryIndexes:
              - IndexName: time-index
                KeySchema:
                  - AttributeName: email
                    KeyType: HASH
                  - AttributeName: time
                    KeyType: RANGE
                Projection:
                  ProjectionType: KEYS_ONLY
            ProvisionedThroughput:
              ReadCapacityUnits: 1
              WriteCapacityUnits: 1
```
## Steps to run the serverless templates.
!!! Note
    If you don't have the serverless framework installed yet, you need to follow the next steps first:
    https://www.serverless.com/framework/docs/getting-started/
    
1. Create a user and configure necessary permissions to run the template.
2. Generate the access keys for created user. 
3. Use those credentials as config for serverless:     
```
serverless config credentials --provider aws --key [key] --secret [secret]
```
4. Download the public repository https://github.com/WilliamPossos/sqs-send-email
5. Run commands
```
make
```
&
```
make build
```
   Those commands use the Makefile content to get the project dependencies and build it. That is to create the goland file that is finally deployed into lambda.
6. Run command
```
serverless deploy -v
```
7. Download the public repository https://github.com/WilliamPossos/sign-in-up-service
8. Repeat steps 5 and 6 into the new repository.
9. Finally, you need to replace https://github.com/WilliamPossos/angular-basic-sign-up/blob/master/src/app/user-service.service.ts this file with the new API Gateway URL (Generated by the serverless template).
After that, the Code Pipeline updates the front-end implementation with the new "server" url.  
10. If necessary, you need to register your email in the SES (it needs to be done in the region where the serverless template was executed). 
