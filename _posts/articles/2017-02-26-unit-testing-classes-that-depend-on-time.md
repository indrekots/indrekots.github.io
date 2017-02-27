---
layout: post
title: "Unit testing classes that depend on time"
excerpt:
modified: 2017-02-26 20:24:27 +0200
categories: articles
tags: [java, time, clock, testing, dependency injection]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
aging: true
---

Sometimes in unit testing you need to have control of the current time. But when your code that's under test depends on the current system time, it's not always that easy to achieve. As developers, we're taught to favor loosely coupled code. I'm sure you've heard of the [Dependency Inversion principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle). It states that you shouldn't depend upon concrete classes but you should depend upon abstractions. Therefore time can also be considered as a dependency.

## Current time as a dependency

Consider the following case. You design a class which needs to check the current time. Let's say you're working on a message board (e.g. Discourse) and you would like to congratulate the user on his or her birthday when they log in. The class would need to check the current date and compare it against the user's entered birthday. If they're equal, display birthday congratulations.

But how would you unit test that class? If your implementation retrieves the current system time, then you have tightly coupled your class to the server's clock. To avoid that, think of the current time as one of many dependencies your class has. You can use [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection "Dependency Injection") to inject a replaceable clock to your class. In a production environment, your class can use a clock that retrieves the current system time. In unit tests, you can pass in a clock that reports a given fixed time.

## `java.time.Clock`

Java 8 introduced a new date and time API. It includes a [Clock](https://docs.oracle.com/javase/8/docs/api/java/time/Clock.html) class which is specifically designed to be used as a dependency.

> Instances of this class are used to find the current instant, which can be interpreted using the stored time-zone to find the current date and time. As such, a clock can be used instead of `System.currentTimeMillis()` and `TimeZone.getDefault()`.

> The primary purpose of this abstraction is to allow alternate clocks to be plugged in as and when required. Applications use an object to obtain the current time rather than a static method. This can simplify testing.

## Example

* code example of clock being injected

* joda time set current time millis example
