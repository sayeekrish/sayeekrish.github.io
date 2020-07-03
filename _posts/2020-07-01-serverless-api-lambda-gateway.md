---
layout: post
author: SaiKrishna Mallupattu
title: Build a REST API in Kotlin with AWS Lambda, API Gateway, DynamoDB and Serverless Framework | Amazon Web Services
---

In this tutorial, I am going to walk you through the steps involved to build an REST api in combination with serverless framework, aws lambda, api gateway and DynamoDB. Before getting into the details, lets understand why should we choose building an API in this way?

There are two kinds of people: consumers and producers.
Consumers always have problems &#128512; & needs whereas Producers sees these problems as opportunities and comes with solutions. Now what happpened over a period of time, we have more solutions than problems.

Now comes the options. Which option to choose among all these solutions?. Building
 an API with api gateway and lambda is one such option among many. This has both advantages and disadvantages.

###### Advantages:

    1. Inherently Scalable
    2. Easy to deploy
    3. No Server management
    4. Cost Effective in unpredictable workload

###### Disadvantages:
    1. Not for long running processes.
    2. Vendor lock-in

With api gateway & lambda combination, We can design our api's in different ways.I will demonstrate one among them; Having a seperate function for each Http Method(POST, PUT, GET, DELETE).

###### Step 1: Install NodeJs
Follow [nodejs-download-page]

###### Step 2: Install Serverless 
Run the below command to install serverless globally.
```cmd 
npm install serverless -g
```
###### Step 3: Create a gradle project
To create a gradle project with template for aws-kotlin-jvm-gradle
```cmd 
serverless create --template aws-kotlin-jvm-gradle --path todo-api
```
###### Step 4: Serverless Configuration
Import the gradle project into an IDE and open `serverless.yml`. You will see something like below removing all commented lines.

```yaml
service: todo-api
provider:
  name: aws
  runtime: java11
package:
  artifact: build/libs/hello-dev-all.jar

functions:
  hello:
    handler: com.serverless.Handler

```

We are going to push the artifact as a zip file. So update the artifact path and filename like below.

```yaml
service: todo-api
provider:
  name: aws
  runtime: java11
package:
  artifact: build/distributions/todo-api.zip

functions:
  hello:
    handler: com.serverless.Handler

```












[nodeJs-download-page]: https://nodejs.org/en/download