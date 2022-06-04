---
layout: post
title:  "Skinny versus Fat Lambdas"
tags: [ AWS, lambda, serverless, apigateway]
featured_image: assets/images/posts/2022/kelly-sikkema--nz-GTuvyBw-unsplash.jpg
featured: true
published: true  
---

Serverless technoligies are incredibly empowering, but the serverless ecosystem lacks opinionated application frameworks. Web application frameworks like [Ruby on Rails](https://rubyonrails.org/) come with a set of conventions reduce the barriers of entry for new and experienced developers alike. This stands in contrast to the serverless ecosystem, where solutions to common application concerns are harder to come by.

For example, consider this discussion on how to design an API using AWS Lambda.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">🤔 I’m still kinda weirded out by this seemingly preferred model of creating a Lambda per http route<br><br>Instead of creating an app and passing through all web requests to one lambda function</p>&mdash; Chris Fidao (@fideloper) <a href="https://twitter.com/fideloper/status/1496506610197409798?ref_src=twsrc%5Etfw">February 23, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Chris is describing what we call "skinny versus fat" Lambdas.  In other words, should an API be implemented with a single Lambda function containing logic for all routes (fat) or a Lambda per route (skinny)?  Maybe some combination of these approaches?  

Here's my take on the tradeoffs of using skinny versus fat lambdas.

### Benefits of Skinny Lambdas

#### Shorter Cold Starts

The deployment size of a lambda function has a direct impact on cold start performance. Bundling only what you need for each route of your API will result in a smaller deployment artifact, resulting in shorter cold starts.

#### Debugging is Easier

Single responsability Lambdas are easier to debug. An error on your `GetUserLambdaFunction` is *much* more meaningful than an error in your monolithic lambda with dozens of routes.

#### Fine Grained Permissions

When you create a Lambda, you assign an AWS Identity and Access Management (IAM) role that grants the function permission to access AWS services and resources. Single responsibility lambdas make it possible to apply the principles of least privelage to each route of your API.

#### Concurrency Controls/Scalability per Function

Lambda functions can be tailored to meet the specific needs of your application.  While it can be useful to independently configure the scaling charactersits of a single function, this approach can make it harder to limit the lambda concurrency across your entire API. 

#### Higher Potential for Code Reuse

Lambda functions can be utilized in numerous contexts; as targets of EventBridge rules, SQS queue processing and Step Function workflows to name a few.  Designing your Lambdas to have single responsabilities could allow you to more easily reuse that Lambda elsewhere in your application.

### Benefits of Fat Lambdas

#### Rarely used functions wont cold start

One of the benefits of a fat lambda is that lesser used routes will rarely have cold starts.

#### Treat Lambda as a Deployment Target

Building an API in a single Lambda function lets you treat Lambda as another deployment target. This is convenient if you ever decide to deploy your API in a container. This approach would also make it easier to run your API in your local environment during development.

#### Easier Mental Shift

APIs are historically built as single, monolith applications. Fat functions mimic this approach, making it more acessible to a broader audience. 


The great news is that *either* approach will work when building APIs on AWS Lambda. The "best" approach is largely a matter of personal preference.  Which approach do you prefer?