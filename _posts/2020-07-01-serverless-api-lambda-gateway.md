---
layout: post
author: SaiKrishna Mallupattu
title: Build Serverless API with AWS Lambda and API Gateway | Amazon Web Services
---

In this post, I am going to walk you through the steps involved to build an api in combination with serverless framework, aws lambda and api gateway. Before getting 
into details, lets understand why should we choose building an API in this way?

In this globalization era, there are two kinds of people: consumers and producers.
Consumers always have problems &#128512; & needs whereas Producers sees these problems as opportunities and comes with solutions. Now what happpened over a period of time, we have more solutions than problems.

Now comes the options. Which option to choose among all these solutions?. Building
 an API with api gateway and lambda is one such option among many. This has both advantages and disadvantages. advantages are plenty though.

1. Scalability & Availability
2. Suitable when your api traffic is difficult to predict and control
3. Easy to deploy
4. Language of your choice

Again in this api gateway and lambda combination, we can build our api's in many ways. I am going to show you **option 2**

1. Single lambda function handling all HTTP methods(POST, PUT, GET, DELETE)
2. Seperate lambda function for each HTTP method.

```yaml
service: posts-api
provider:
  name: aws
  runtime: java11
  profile: serverless
```
```java
public void invoke() {
        OkHttpClient httpClient = new OkHttpClient();
        Request request = new Request.Builder()
                .url(System.getenv("ENDPOINT"))
                .addHeader("Authorization", Credentials.basic(System.getenv("user"), System.getenv("pass")))
                .build();
        try (Response response = httpClient.newCall(request).execute()) {
            if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```








