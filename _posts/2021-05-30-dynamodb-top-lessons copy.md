---
layout: post
title:  "Things I Wish I Knew Earlier: Top 5 Lessons from working with DynamoDB"
tags: [ AWS, Lambda, Node, JavaScript, Asyncronous]
featured_image: assets/images/posts/2021/tobias-fischer-PkbZahEG2Ng-unsplash.jpg
featured: false
published: false  
---

## NoSQL != non-relational

The greatest misconcenption of NoSQL databases is that they are not able to model relational data. This is an unfortunate side effect of the branding of SQL and NoSQL databases as relational and non-relational.

Let's explore this idea by showing how we could model a basic blog application with Posts, Comments and Authors.  Here's how the entities relate to one another.

![Blog ER Diagram](../assets/images/posts/2021/blog_erd.png#center)
 

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">NoSQL is not non-relational. All data is relational or nobody would care about it. NoSQL is denormalized. Entity Relationship Models still matter.</p>&mdash; Rick Houlihan (@houlihan_rick) <a href="https://twitter.com/houlihan_rick/status/1084943494282657792?ref_src=twsrc%5Etfw">January 14, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

A more meaningful distinction between SQL and NoSQL databses is normalized vs. denormalized. SQL databases are normalized to optomize for space efficiency where NoSQL databases are denormalized to optomize for compute efficiency.  


## It's All About the Access Patterns


## DynamoDB is not a _flexible_ database, it's an **efficient** database

## Primary Key Design is Critical

Create collections of items interesting to the application, create key conditions and filter conditions that return items that are interesting to the application.
