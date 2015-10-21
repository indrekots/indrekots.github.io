---
layout: post
title: "Java 8: Behavior parameterization"
excerpt:
modified: 2015-10-17 21:55:09 +0300
categories: articles
tags: [java8, java]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
---

Behavior parameterization is essentially taking a block of code and making it available without executing it. It can be passed to a method for example. Since Java 8 introduced lambdas (finally), it is now possible to parameterize method's behavior. If you're familiar with Javascript, Scala, Groovy or any other language that has lambdas built in, then you probably have been using behavior parameterization all along.

It might not come to you as a surprise that in software development user requirements can change. Passing behavior as a parameter can help relieve the pain of change.

##Filter with loop and if clause
###show boilerplate code

##Introduce predicates, filter clause in a separate Object
###Multiple implementations
###anonymous classes

##Replace predicate function with a lambda

##Use streams and filter method (more functional style)
###parallelization, no loops

##useful for flexible apis
