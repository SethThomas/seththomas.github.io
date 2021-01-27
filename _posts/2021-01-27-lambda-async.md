---
layout: post
title:  "The Async Lambda Trap"
tags: [ AWS, Lambda, Node, JavaScript, Asyncronous]
featured_image: assets/images/posts/2020/amirali-mirhashemian-kiH-RBm08NQ-unsplash.jpg
featured: true
published: true  
---

Serverless computing enables us to execute our code without the burden of provisioning or managing servers.  This simple yet powerful paradigm enables us to build all sorts of cool things _quickly_. However, we aren't _entirely_ free of the burden of understanding whats going on "under the hood".

In this blog post, I'll briefly explain the [AWS Lambda](https://aws.amazon.com/lambda/) execution environment and illustrate how you can avoid one of the most common pitfalls when using the popular service.

### The Problem

Let's start by looking at a simple Lambda function that stores data in DynamoDB.

```JavaScript
var AWS = require('aws-sdk');
var ddb = new AWS.DynamoDB.DocumentClient();

export async function main(event, context) {

  let body = JSON.parse(event.body);

  var params = {
      TableName: 'my_table',
      Item: { message : body.message }
  };

  ddb.put(params, function(err,data){
    if (err) console.log("Oh no, an error!");
    else     console.log("Hooray, it worked!");
  });

  return {statusCode: 200};
}
```

This lambda parses the event body, extracts the parameter named `message` and sends it off to DynamoDB for storage.

After executing this lambda and getting a 200 response, you are surprised to find no update to DynamoDB. You head over to the logs and check for any error messages, only to find nothing. The lambda was executed but you see no logs from the callbacks of the `ddb.put` operation.  You practically copy/pasted the code directly from the [DynamoDB docs](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#put-property), so you know you're calling the `put` operation correctly.

To make matters worse, you execute the lambda several more times only to see it eventually work!  Well, not consistently, but it _does_ work sometimes and not others.  

Ugh.  What. The. Eff?!

### Understanding the Lambda Execution Lifecycle

To better understand what's going on here, let's take a brief look at the [Lambda execution environment lifecycle](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-context.html), which consists of three distinct phases:

* **Init** - Lambda creates or un freezes an execution environment, downloads the code for the function, initializes extensions, initializes the runtime, etc.  
* **Invoke** - This phase is when Lambda runs your code.  After the function runs to completion, Lambda prepares to handle another function invocation.
* **Shutdown** - If the Lambda function does not receive any invokes for a period of time, it terminates the runtime an removes the environment.

Building the temporary execution environment takes time. To improve performance, Lambda will try to reuse the execution environment for subsequent invocations.  This will lead to fewer cold starts, make your application more responsive _and_ save you money.  However, reusing the execution environment can lead to unexpected behavior.  The docs highlight this _allllll_ the way near the bottom of the page

> Background processes or callbacks that were initiated by your Lambda function and did not complete when the function ended resume if Lambda reuses the execution environment

This, in a nutshell, is our problem!

### The Solution

The problem is that our lambda is executing our code _asynchronously_ and our non-blocking call to `ddb.put` never gets a chance to complete.  As a result, our callbacks never get called.

Because our lambda execution environment *might* be reused, the call to `ddb.put` *might* be allowed to complete.

Luckily, the solution is fairly straightforward.  We need to tell the lambda to wait for the response of our call to DynamoDB.  This can be accomplished by using the `asnyc/await` syntax:

```JavaScript
var AWS = require('aws-sdk');
var ddb = new AWS.DynamoDB.DocumentClient();

export async function main(event, context) {

  let body = JSON.parse(event.body);

  var params = {
      TableName: 'my_table',
      Item: { message : body.message }
  };

  try{
    await ddb.put(params).promise();
    console.log("Hooray, it worked!");
  } catch(err){
    console.log("Oh no, an error!");
  }

  return {statusCode: 200};
}
```


### Conclusion

Serverless compute frees us from the headache of managing server-side infrastructure, but we still need to be mindful of the execution environment.  I used the DynamoDB SDK in this post to highlight a real-world scenario that I see pop up [all](https://stackoverflow.com/questions/64650275/how-to-put-items-into-dynamodb-table-using-lambdanode-js/64650717#64650717) [the](https://stackoverflow.com/questions/65911314/lambda-functions-getitem-call-in-nodejs-is-not-printing-any-item-in-console/65911858#65911858) [time](https://stackoverflow.com/questions/55920683/lambda-function-is-not-calling-https-request-function-when-data-is-retrieved-fr). However, this problem is not unique to the DynamoDB SDK.  Rather, it's something we need to be aware of anytime we use background processes or callbacks in asynchronous lambdas.


TLDR; When using Lambda async handlers, always make sure any background processes or callbacks are complete before the code exists.  
