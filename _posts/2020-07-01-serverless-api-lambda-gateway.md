---
layout: post
author: SaiKrishna Mallupattu
title: Build a REST API in Kotlin with AWS Lambda, API Gateway, DynamoDB and Serverless Framework | Amazon Web Services
---

In this tutorial, I am going to walk you through the steps involved to build an REST api in combination with serverless framework, aws lambda, api gateway and DynamoDB. Here are few advantages and disadvantages doing it this way.

###### Advantages:

    1. Inherently Scalable
    2. Easy to deploy 
    3. No Server management
    4. Cost Effective in unpredictable workload

###### Disadvantages:
    1. Not for long running processes.
    2. Vendor lock-in

I am taking a simple usecase here, an todo app to show you basic operations like create, get, update and delete an task.

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

We are going to have 5 functions in total. List them under functions section

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
  todo-put:
    handler: in.co.learndesk.todos.TodoRequestHandler::handlePutRequest
  todo-get:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleGetRequest
  todo-get-all:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleGetAllRequest
  todo-delete:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleDeleteRequest
```

Now attach http endpoint for each of these functions along with its Http method

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
  todo-put:
    handler: in.co.learndesk.todos.TodoRequestHandler::handlePutRequest
    events:
      - http:
          cors: true
          path: user/{userId}/todos/{todoId}
          integration: lambda_proxy
          method: post
  todo-get:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleGetRequest
    events:
      - http:
          cors: true
          path: user/{userId}/todos/{todoId}
          integration: lambda_proxy
          method: get
  todo-get-all:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleGetAllRequest
    events:
      - http:
          cors: true
          path: user/{userId}/todos
          integration: lambda_proxy
          method: get
  todo-delete:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleDeleteRequest
    events:
      - http:
          cors: true
          path: user/{userId}/todos/{todoId}
          integration: lambda_proxy
          method: delete
```

For post and put method, generally content-type is mandatory. So setting the content-type header as mandatory

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
  todo-put:
    handler: in.co.learndesk.todos.TodoRequestHandler::handlePutRequest
    events:
      - http:
          cors: true
          path: user/{userId}/todos/{todoId}
          integration: lambda_proxy
          method: post
          request:
            parameters:
              headers:
                Content-Type: true
```

For tracking and debugging purposes, I am introducing a header in request and also making it mandatory

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
todo-put:
    handler: in.co.learndesk.todos.TodoRequestHandler::handlePutRequest
    events:
      - http:
          cors: true
          path: user/{userId}/todos/{todoId}
          integration: lambda_proxy
          method: post
          request:
            parameters:
              headers:
                Content-Type: true          
                appRequestId: true
  todo-get:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleGetRequest
    events:
      - http:
          cors: true
          path: user/{userId}/todos/{todoId}
          integration: lambda_proxy
          method: get
          request:
            parameters:
              headers:
                appRequestId: true
  todo-get-all:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleGetAllRequest
    events:
      - http:
          cors: true
          path: user/{userId}/todos
          integration: lambda_proxy
          method: get
          request:
            parameters:
              headers:
                appRequestId: true
  todo-delete:
    handler: in.co.learndesk.todos.TodoRequestHandler::handleDeleteRequest
    events:
      - http:
          cors: true
          path: user/{userId}/todos/{todoId}
          integration: lambda_proxy
          method: delete
          request:
            parameters:
              headers:
                appRequestId: true
```
