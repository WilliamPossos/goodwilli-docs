# Cloud challenge - Stage 3
## Site URL
You can use the following url to see the solution site:
```
https://challenge-3.goodwilli.com/
```
## Solution architecture
![AWS architecture](https://joiners-challenge.s3.amazonaws.com/aws-challenge-3.jpeg)

The components are:

- Angular project to build the solution Front-End.
- Static website to host the built front-end (AWS s3).
- AWS CLoudFront to deliver **securely** the s3 content.
- Api gateway acts as the "front door" for applications to access data, business logic, or functionality from your backend services.
- Bank-end lambda used to manage the backend flow of sign in and sign up.
- Simple Queue Service (SQS) service to queue the messages which will then be sent to users emails.
- Lambda triggered by SQS and uses SES service to send the emails.
- Simple Email Service (SES) service that enables email sending.
    
## Continuous integration and delivery
Once the code has been pushed to github repository. The Code Pipeline of aws starts the automatic build and deploy of our front-end/back-end code

Important:
- Build specification reference for CodeBuild: A buildspec is a collection of build commands and related settings, in YAML format, that CodeBuild uses to run a build. You can include a buildspec as part of the source code or you can define a buildspec when you create a build project. (Examples `buildspec_debug.yml`, `buildspec_release.yml` and `arn:aws:s3:::my-codebuild-sample2/buildspec.yml`).
## Related documentation:
- **Buildspec:** https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html
