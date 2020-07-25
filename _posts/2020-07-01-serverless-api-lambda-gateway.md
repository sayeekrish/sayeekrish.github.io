---
layout: post
author: SaiKrishna Mallupattu
title: Build a REST API in Kotlin with AWS Lambda, API Gateway, DynamoDB and Serverless Framework | Amazon Web Services
---

In this post, I am going to show you the steps involved to build an REST api in combination with serverless framework, aws lambda, api gateway and DynamoDB. Here are few advantages and disadvantages doing it this way.

###### Advantages:

    1. Inherently Scalable
    2. Easy to deploy 
    3. No Server management
    4. Cost Effective in unpredictable workload

###### Disadvantages:
    1. Not for long running processes (30 sec is the limit for the lambda service when you use along with API Gateway)
    2. Vendor lock-in.

I am taking a simple usecase here, a post API to add a task, full implementation of this article can be found in the github project

###### Software Requirements:
    1. Nodejs
    2. Serverless
    3. Gradle
    4. Kotlin
    5. Java
    6. AWS Cli

###### Step 1: Create a gradle project
To create a gradle project with template for aws-kotlin-jvm-gradle
```cmd 
serverless create --template aws-kotlin-jvm-gradle --path todos
```
###### Step 2: Serverless Configuration
Import the gradle project into an IDE and open `serverless.yml`. You will see something like below after removing all commented lines.

```yaml
service: todos
provider:
  name: aws
  runtime: java11
package:
  artifact: build/libs/hello-dev-all.jar

functions:
  hello:
    handler: com.serverless.Handler

```

We are going to bundle our artifact as a zip file. So update the artifact path and filename like below.

```yaml
service: todos
provider:
  name: aws
  runtime: java11
package:
  artifact: build/distributions/todos.zip

functions:
  hello:
    handler: com.serverless.Handler

```

We are going to have 5 API's. Each API will have its own function. List them all under functions section.

```yaml
service: todos
provider:
  name: aws
  runtime: java11
package:
  artifact: build/distributions/todos.zip

functions:
  todo-post:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleRequest
```

Now attach an http endpoint for each of these functions along with its Http method

```yaml
functions:
  todo-post:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleRequest
    events:
      - http:
          cors: true
          path: user/{userId}/todo
          integration: lambda_proxy
          method: post
```

Generally content-type is mandatory for all post methods. So setting the content-type header as mandatory

```yaml
functions:
  todo-post:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleRequest
    events:
      - http:
          cors: true
          path: user/{userId}/todo
          integration: lambda_proxy
          method: post
          request:
            parameters:
              headers:
                Content-Type: true
```

For tracking purpose, I am introducing a header to the request and response and making it as mandatory

```yaml
functions:
  todo-post:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleRequest
    events:
      - http:
          cors: true
          path: user/{userId}/todo
          integration: lambda_proxy
          method: post
          request:
            parameters:
              headers:
                Content-Type: true
                appRequestId: true
```

We have to take extra care when dealing with post and put methods. These methods generally carry request body and we must validate it. We can't do all this inside a lambda function. Doing it this way will also incur more cost from us. Because even for false request, our lambda function is being invoked and executed. API Gateway will help us to validate request body, path parameters and allow only well structured requests to the lambda service. 

This requires you to install `serverless-reqvalidator-plugin` an npm module to make use of validator on method.

```cmd 
npm i serverless-reqvalidator-plugin
```

In serverless.yml specify the installed plugin under plugins section

```yaml
plugins:
  - serverless-reqvalidator-plugin
```

In serverless.yml create custom resource for request validators


```yaml
resources:
  Resources:
    TodoRequestValidator:
      Type: "AWS::ApiGateway::RequestValidator"
      Properties:
        Name: 'todo-request-validator'
        RestApiId:  !Ref ApiGatewayRestApi
        ValidateRequestBody: true
        ValidateRequestParameters: true
```

For every function you wish to use the validator set property `reqValidatorName: TodoRequestValidator` to match resource you described

```yaml
functions:
  todo-post:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleRequest
    events:
      - http:
          cors: true
          path: user/{userId}/todo
          integration: lambda_proxy
          method: post
          reqValidatorName: TodoRequestValidator
          request:
            parameters:
              headers:
                Content-Type: true
                appRequestId: true
```

Now comes the model definition on how the request body should be. We are going to use an another serverless plugin `serverless-aws-documentation` that adds support for documentation and models

```cmd 
npm i serverless-aws-documentation
```

Next, add the serverless-aws-documenation plugin into serverless.yml file:

```yaml
plugins:
  - serverless-reqvalidator-plugin
  - serverless-aws-documentation
```

Define the model

```yaml
custom:
  documentation:
    api:
      info:
        version: '1.0.0'
        title: Todo app
    models:
      - name: TodoRequestModel
        contentType: application/json
        schema:
          type: object
          properties:
            name:
              type: string
            isComplete:
              type: boolean           
            category:
              type: array
              items:
                type: string
                uniqueItems: true
                minItems: 1
                enum: ["HOME", "WORK"]
          required: [name, isComplete, category]
```

Next add the model on to the method

```yaml
functions:
  todo-post:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleRequest
    events:
      - http:
          documentation:
            summary: Create an item
            description: Add a new item to the todo list
            requestModels:
              "application/json": TodoRequestModel
          cors: true
          path: user/{userId}/todo
          integration: lambda_proxy
          method: post
          reqValidatorName: TodoRequestValidator
          request:
            parameters:
              headers:
                Content-Type: true
                appRequestId: true
```
Now we have given full responsibility to gateway service to validate our API's before it calls our lambda function. API Gateway has default template defined, But We can customize to our needs. Also as I said at the begining, We should add appResponseId header to trace the response. Here is the place to do it. Add the below part to the existing resources section:

```yaml
    Unauthorized:
      Type: "AWS::ApiGateway::GatewayResponse"
      Properties:
        ResponseParameters:
          gatewayresponse.header.appResponseId: method.request.header.appResponseId
        ResponseTemplates:
          "application/json": '{"responseHeader":{"statusCode":401,"statusMessage":"Requested resource is restricted and requires authentication"}}'
        ResponseType: UNAUTHORIZED
        RestApiId:  !Ref ApiGatewayRestApi
        StatusCode: 401
    Forbidden:
      Type: "AWS::ApiGateway::GatewayResponse"
      Properties:
        ResponseParameters:
          gatewayresponse.header.appResponseId: method.request.header.appResponseId
        ResponseTemplates:
          "application/json": '{"responseHeader":{"statusCode":403,"statusMessage":"User is not authorized to access this resource"}}'
        ResponseType: ACCESS_DENIED
        RestApiId:  !Ref ApiGatewayRestApi
        StatusCode: 403
    BadRequest:
      Type: "AWS::ApiGateway::GatewayResponse"
      Properties:
        ResponseParameters:
          gatewayresponse.header.appResponseId: method.request.header.appResponseId
        ResponseTemplates:
          "application/json": '{"responseHeader":{"statusCode":400,"statusMessage":"Request was malformed or syntactically incorrect"}}'
        ResponseType: BAD_REQUEST_BODY
        RestApiId:  !Ref ApiGatewayRestApi
        StatusCode: 400
    BadRequestParameters:
      Type: "AWS::ApiGateway::GatewayResponse"
      Properties:
        ResponseParameters:
          gatewayresponse.header.appResponseId: method.request.header.appResponseId
        ResponseTemplates:
          "application/json": '{"responseHeader":{"statusCode":400,"statusMessage":"Request was malformed or syntactically incorrect"}}'
        ResponseType: BAD_REQUEST_PARAMETERS
        RestApiId:  !Ref ApiGatewayRestApi
        StatusCode: 400
    ResourceNotFound:
      Type: "AWS::ApiGateway::GatewayResponse"
      Properties:
        ResponseParameters:
          gatewayresponse.header.appResponseId: method.request.header.appResponseId
        ResponseTemplates:
          "application/json": '{"responseHeader":{"statusCode":404,"statusMessage":"Malformed or unknown URI"}}'
        ResponseType: RESOURCE_NOT_FOUND
        RestApiId:  !Ref ApiGatewayRestApi
        StatusCode: 404
```
