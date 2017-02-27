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

Sometimes in unit testing you need to have control of the current time. But when you're code that's under test depends on system time, it's not always easy to achieve. As developers, we're taught to favor loosely coupled code. I'm sure you've heard of the [Dependency Inversion principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle). It states that you shouldn't depend upon concrete classes but you should depend upon abstractions. Therefore time should also be a dependency.

* replaceable clock instead of system clock
* time is a dependency, your class needs to know the time so it can work properly, this is from the outside world and should be considered as a dependency
* you can use familiar techniques to inject a reference to the current/test time to your class, e.g. dependency injection
* use real clock in production
* in unit tests, pass a fixed clock, or mock it

* java.time.Clock -> javadoc -> to allow alternate clocks to be plugged in as and when required

* joda time set current time millis example

* code example of clock being injected
