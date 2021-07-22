# Cloud challenge - Stage 3
For this stage, the serverless framework has been applied.

## Background
Develop, deploy, troubleshoot and secure your serverless applications with radically less overhead and cost by using the Serverless Framework.
The Serverless Framework consists of an open source CLI and a hosted dashboard.
Together, they provide you with full serverless application lifecycle management.

When we are working with the serverless framework, we can create a template using different file formats. For this documentation and examples, the YAML format was used.

## Templates used for solution approach
The proposed solution here uses two lambda functions, around which the serverless templates will center.

## SQS - Lambda - SES
Important notes:

- `service:` is like a project. It's where you define your functions, events, resources and so on.
- `provider:` allows to configure the cloud provider (AWS, Azure, Google, etc.). Also, runtime, stage and region
- `provider.deploymentBucket:` allows to configure the bucket used to load some required files for run the cloudformation stack.

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
