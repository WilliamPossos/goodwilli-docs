# Cloud challenge - Stage 3
## Site URL
You can use the following url to see the solution site:
```
https://challenge-3.goodwilli.com/
```
## Proposed architecture
![AWS architecture](https://joiners-challenge.s3.amazonaws.com/aws-challenge-3.jpeg)

The components are:

- Angular project to build the solution Front-End.
- Static website to host the built front-end (AWS s3).
- AWS CloudFront to deliver **securely** the s3 content.
- Api gateway acts as the "front door" for applications to access data, business logic, or functionality from your backend services.
- Bank-end lambda used to manage the backend flow of sign in and sign up.
- Simple Queue Service (SQS) service to queue the messages which will then be sent to users emails.
- Lambda triggered by SQS and uses SES service to send the emails.
- Simple Email Service (SES) service to enable email sending.

## Lambda (Golang) + Api Gateway
In Lambda proxy integration, the input to the integrated Lambda function can be expressed as any combination of request headers, path variables, query string parameters, and body. In addition, the Lambda function can use API configuration settings to influence its execution logic. For an API developer, setting up a Lambda proxy integration is simple. Other than choosing a particular Lambda function in a given region, you have little else to do. API Gateway configures the integration request and integration response for you. Once set up, the integrated API method can evolve with the backend without modifying the existing settings. This is possible because the backend Lambda function developer parses the incoming request data and responds with desired results to the client when nothing goes wrong or responds with error messages when anything goes wrong.
### Gin
Gin is a web framework written in Go (Golang)
### Lambda Go Api Proxy
Makes it easy to run Golang APIs written with frameworks such as Gin with AWS Lambda and Amazon API Gateway

### Create and configure with aws web console
1. ![Lambda list](https://joiners-challenge.s3.amazonaws.com/lambda-api-gw/lambda-list-view.png)
1. ![Create lambda view](https://joiners-challenge.s3.amazonaws.com/lambda-api-gw/create-lambda-go-1.png)
1. ![Lambda main view](https://joiners-challenge.s3.amazonaws.com/lambda-api-gw/lambda-main-view.png)
1. ![Linking trigger api gateway](https://joiners-challenge.s3.amazonaws.com/lambda-api-gw/lambda-add-trigger-api-gw-1.png)
1. ![Linking trigger api gateway](https://joiners-challenge.s3.amazonaws.com/lambda-api-gw/lambda-add-trigger-api-gw-2.png)
1. ![Create resource/method](https://joiners-challenge.s3.amazonaws.com/lambda-api-gw/configure-api-gw-resources-1.png)
1. ![Create resource/method](https://joiners-challenge.s3.amazonaws.com/lambda-api-gw/configure-api-gw-resources-2.png)
1. ![Create resource/method](https://joiners-challenge.s3.amazonaws.com/lambda-api-gw/configure-api-gw-resources-3.png)
1. ![Linking lambda](https://joiners-challenge.s3.amazonaws.com/lambda-api-gw/configure-api-gw-link-lambda.png)
1. ![Crating stage](https://joiners-challenge.s3.amazonaws.com/lambda-api-gw/configure-api-gw-stage-1.png)
1. ![Crating stage](https://joiners-challenge.s3.amazonaws.com/lambda-api-gw/configure-api-gw-stage-2.png)
1. ![Deploying API](https://joiners-challenge.s3.amazonaws.com/lambda-api-gw/configure-api-gw-deploy.png)

## Using aws Cloudformation
AWS CloudFormation gives you an easy way to model a collection of related AWS and third-party resources, provision them quickly and consistently, and manage them throughout their lifecycles, by treating infrastructure as code

## Continuous integration and delivery
Once the code has been pushed to github repository. The Code Pipeline of aws starts the automatic build and deploy of our front-end/back-end code

Important:
- Build specification reference for CodeBuild: A buildspec is a collection of build commands and related settings, in YAML format, that CodeBuild uses to run a build. You can include a buildspec as part of the source code or you can define a buildspec when you create a build project. (Examples `buildspec_debug.yml`, `buildspec_release.yml` and `arn:aws:s3:::my-codebuild-sample2/buildspec.yml`).

Examples:

Build specification to deploy angular project in s3
```
version: 0.1
env:
  variables:
    S3_BUCKET: "possos-challenge-3"
phases:
  pre_build:
    commands:
      - echo Installing source NPM dependencies...
      - npm install
      - npm install -g @angular/cli
  build:
    commands:
      - echo Build started on `date`
      - ng build --prod --aot
  post_build:
    commands:
      - aws s3 cp dist s3://possos-challenge-3 --recursive
      - echo Build completed on `date`
artifacts:
  files:
    - '**/*'
  base-directory: 'dist*'
  discard-paths: no
```

Build specification to deploy lambda (Go)
```
version: 0.2

phases:
  install:
    runtime-versions:
      golang: 1.15
  build:
    commands:
      - cd $CODEBUILD_SRC_DIR
      - echo Downloading Go packages...
      - go get -u ./...
      - echo Testing Go code...
      - go test
      - echo Building the Go code...
      - GOOS=linux GOARCH=amd64 go build -o application application.go
      - zip application.zip application
      - aws lambda update-function-code --function-name sign-up --zip-file fileb://$CODEBUILD_SRC_DIR/application.zip
artifacts:
  files:
    - application
    - appspec.yml
```

Build specification to deploy a static website (MkDocs)
```
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.8
  pre_build:
    commands:
      - pip install --upgrade pip
      - pip install mkdocs
  build:
    commands:
      - echo building site...
      - mkdocs --version
      - mkdocs build
artifacts:
  files:
    - '**/*'
  base-directory: 'site'
```

## Related documentation:
- **Create a pipeline that uses Amazon S3 as a deployment provider:** https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-s3deploy.html
- **CloudFront to serve a static website:** https://aws.amazon.com/es/premiumsupport/knowledge-center/cloudfront-serve-static-website/
- **Validate domain ownership:** https://docs.aws.amazon.com/acm/latest/userguide/dns-validation.html
- **Domain validation using DNS with AWS Certificate Manager:** https://aws.amazon.com/es/blogs/security/easier-certificate-validation-using-dns-with-aws-certificate-manager/
- **Buildspec:** https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html


Lambda / Gin

- **Gin to build web server (Go):** https://github.com/gin-gonic/gin
- **Go API proxy:** https://github.com/awslabs/aws-lambda-go-api-proxy
- **Build an API Gateway REST API with Lambda integration:** https://docs.aws.amazon.com/apigateway/latest/developerguide/getting-started-with-lambda-integration.html
