---
layout: post
title: "Getting started with Mockito"
excerpt: "A tutorial for getting your feet of the ground with Mockito"
modified: 2013-12-24 13:01:32 +0300
categories: articles
tags: [java, mock, stub, mockito, testing, tutorial]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
published: false
---

[Mockito](http://mockito.org/ "Mockito homepage") is a mocking framework for unit tests written in Java. It lets you write beautiful tests with clean and simple API. This is a straightforward overview of the basic features of Mockito so you can get started with writing clean and readable tests.

Mock objects mimic real and often complex objects. They're used in unit tests instead of actual objects when the actual object is impractical or impossible to instantiate. For example, testing a behavior which depends on an external webservice is non-deterministic and costly to set up in a unit test. Therefore it is a good idea to mock the service so it can respond fast and return expected results.