---
layout: post
title:  "Why Your Concurrent Lambda Isn't Working"
tags: [ AWS, Lambda, Node, JavaScript, Asyncronous]
featured_image: assets/images/posts/2020/amirali-mirhashemian-kiH-RBm08NQ-unsplash.jpg
published: false
---

Writing asynchronous code in Lambda using Node.js is easy.  Using the power of Javascript Promises and the async/await syntactic sugar, we can abandon callback hell, resulting in a more readable and maintainable codebase.  However, there are some common pitfalls that trip people up.  


Consider the following Lambda function that returns a list of buckets from S3.

```Javascript
const AWS = require('aws-sdk')
const s3 = new AWS.S3()

// lambda handler function
exports.handler = async function(event) {

  const buckets = s3.listBuckets().promise()
  return buckets

}
```

Do you think this method

A. returns a list of S3 buckets
B. returns nothing


### A Simple Function

Let's start by looking at a simple Lambda function that uploads a file to S3 and logs the upload in DynamoDB.  Each of these methods is

```JavaScript

exports.handler = async function(event) {

  const s3Result = await asynUploadFileToS3()

  const dbResult = asyncLogUploadToDynamoDB()

  return dbResult;
}
```

### The Lambda Execution Lifecycle

To better understand what's going on here, let's take a brief look at the Lambda execution lifecycle, which consists of three distinct phases:

* **Init** - Lambda creates or un freezes an execution environment, downloads the code for the function, initializes extensions, initializes the runtime, etc.  
* **Invoke** - This phase is when Lambda runs your code.  After the function runs to completion, Lambda prepares to handle another function invocation.
* **Shutdown** - If the Lambda function does not receive any invokes for a period of time, it terminates the runtime an removes the environment.

Building the temporary execution environment takes time. To improve performance, Lambda will try to reuse the execution environment for subsequent invocations.  This will lead to fewer cold starts, make your application more responsive _and_ save you money.  

For example, consider a Lambda function that uploads a file to S3 and logs the file upload to DynamoDB.  
