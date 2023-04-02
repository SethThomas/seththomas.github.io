---
layout: post
title:  "My Love/Hate Relationhip with EventBridge Pipes"
tags: [ AWS, lambda, serverless, eventbridge, pipes]
featured_image: assets/images/posts/2023/sigmund-rVRvR9VUIoQ-unsplash.jpg
featured: true
published: true  
---


I love deleting code. Anytime I hear of a new AWS service that helps me do this, I pay very close attention. When [EventBridge Pipes](https://aws.amazon.com/eventbridge/pipes/) was first announced at the AWS re:Invent, I was immediately smitten.

EventBridge Pipes is a new feature of EventBridge that lets you connect AWS services without needing to write custom application code. 

> Amazon EventBridge Pipes helps you create point-to-point integrations between event producers and consumers with optional transform, filter and enrich steps. EventBridge Pipes reduces the amount of integration code you need to write and maintain when building event-driven applications.

The AWS Console provides an intuitive interface to explain how Pipes work

![EventBridge Pipes UI](../assets/images/posts/2023/pipes-ui.png#center)

SQS and DynamoDB Streams are among the list of supported sources.  This was great news to me, as both of these services are featured prominently in my current projects.  I was excited to see how Pipes could help me delete code!

### Use Case: DynamoDB Steams to EventBridge

I have an application that uses DynamoDB Streams to capture user management events (i.e. UserCreated, UserUpdated, UserDeleted).  Today, my architecture looks like this:


![DynamoDB Streans Architecture Before](../assets/images/posts/2023/ddb-stream-arch1.png#center)

The role of the Lambda Function in the above diagram is to transform the DynamoDB Stream payload into a corresponding event (i.e. UserCreated) before publishing to EventBridge. 

With Pipes, I should be able to remove the Lambda Function to get the following solution:

![DynamoDB Streans Architecture After](../assets/images/posts/2023/ddb-stream-arch2.png#center)

or so I thought...

#### The Problem: Limited JSON Path Support

Without Lambda, I need Pipes to do the work of transforming my DynamoDB formatted JSON into a UserCreated event. 

For example, a DynamoDB Stream payload for a new User item may look like this:

```JSON
{
  "eventID": "c814968f8803051fa5700a2a0b9fe599",
  "eventName": "INSERT",
  "eventVersion": "1.1",
  "eventSource": "aws:dynamodb",
  "awsRegion": "us-east-1",
  "dynamodb": {
    "ApproximateCreationDateTime": 1672592664,
    "Keys": {
      "sk": {
        "S": "PROFILE"
      },
      "pk": {
        "S": "USER#testuser"
      }
    },
    "NewImage": {
      "username": {
        "S": "testuser"
      },
      "roles": {
        "L": [{
          "S": "admin"
        },{
          "S": "otherrole"
        }]
      },
      "SequenceNumber": "3044600000000017324272463",
      "SizeBytes": 505,
      "StreamViewType": "NEW_AND_OLD_IMAGES"
    }
  }
}
```

and I want to convert this to the following EventBridge event payload:


```JSON
{
   data:{
     username: "testuser"
     roles: ["admin","otherrole"]
   },
   metadata:{
     eventType: "UserCreated"
   }
}
```

Unfortunately, I was not able to completely remove the DynamoDB formatting from the `roles` attribute because the targer Input Transformer was unable to work with lists! (believe me, [I tried](https://stackoverflow.com/questions/74976461/eventbridge-pipes-target-input-transformer-formatting-lists-arrays/75587543#75587543)).  

Due to this limitation, the best I could do was the following:

```JSON
{
  "data": {
    "username": "testuser",
    "roles": [{
      "S": "admin"
    },{
      "S": "otherrole"
    }]
  },
  "metadata": {
    "eventType": "UserCreated"
  }
}
```

Since I don't want downstream consumers to know how to handle DynamoDB formatted JSON, this approach is a non-starter for me. Sounds like I'm sticking to my existing solution for the time being.

### Use Case: DynamoDB Streams to SNS FIFO

I have another workload that uses DynamoDB streams to publish events to an SNS Fifo Topic.  I'm using an SNS Fifo Topic to implement a fanout pattern with an ordering guarentee.  The solution looks like this:

![DynamoDB Streans Architecture Before](../assets/images/posts/2023/ddb-streams-sns-fifo.png#center)

Similar to the prior use case, I had a Lambda Function that transforms DynamoDB formatted JSON and publishes the content to the SNS Topic.  

#### The Problem: SNS Fifo Topic != valid Target

I tried building the solution via CDK, which deployed without error.  However, the Pipes AWS Console seemed to suggest something had gone wrong

![Pipes SNS FIFO with CDK](../assets/images/posts/2023/pipe.png#center)


Updating my CDK code to use a _non_ Fifo Topic seemed to fix the issue.

![Pipes SNS with CDK](../assets/images/posts/2023/pipe-fix.png#center)

But... I want a Fifo Topic!

Thinking I must have made a mistake in my CDK code, I tried to configure the Pipe via the AWS Console. Unfortuately, that didn't work either

![Pipes via AWS Console](../assets/images/posts/2023/pipe-console.png#center)

I couldn't seem to find this limitation documented anywhere, but this configuration does not seem to be supported.  

Bummer!


### Now What?

EventBridge Pipes is a _fantastic_ feature of EventBridge. I am confident that the EventBridge team will continue to iterate and add new functionality to this service.

However, my experience with Pipes thus far has left me wanting more. I've come *very* close to being able to delete custom application code, but ultimately had to stick with my current solution due to a missing feature or undocumented limitation.

Nevertheless, I'm bullish on the future of Pipes and will continue to search for use cases for this feature.  If you're working with EventBridge, I'd recommend giving it a look to see if you can delete some code!