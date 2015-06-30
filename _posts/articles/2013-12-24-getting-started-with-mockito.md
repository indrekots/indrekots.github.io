---
layout: post
title: "Getting started with Mockito - a tutorial for beginners"
excerpt: "A tutorial for getting your feet of the ground with Mockito"
modified: 2013-12-24 13:01:32 +0300
categories: articles
tags: [java, mock, stub, mockito, testing, tutorial, maven, gradle]
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

##Getting started

I'm going to assume you have Mockito already in place and ready to go. Whether you got it via a build system (e.g. [Maven](https://maven.apache.org/ "Apache Maven Project homepage") or [Gradle](https://gradle.org/ "Gradle homepage")) or manually, it does not matter.

Fristly, let's look at how to stub a method. We have a user data access object which communicates with a database. In a unit test it is preferred to mock data access to get deterministic results and have a faster running test suite.

To improve code readablity, use a static import	
{% highlight java %}
import static org.mockito.Mockito.*;
{% endhighlight %}

Create a new mock `userDao`

{% highlight java %}
UserDao userDao = mock(UserDao.class);
{% endhighlight %}

Then stub user data access:

{% highlight java %}
when(userDao.getById(1L)).thenReturn(new User("Mario"));
{% endhighlight %}

This almost reads like a sentence, right? Whenever `getById()` is called with `1L`, new user `Mario` is returned.

##Throwing exceptions

What if instead of returning a new user you want to test an edge case. Suppose `userDao` is implemented to throw an exception when a user is not found. Mockito provides a `thenThrow()` method.

{% highlight java %}
when(userDao.getById(1L)).thenThrow(new NoResultException());
{% endhighlight %}

##Keeping internal state
In some cases it might be useful to keep internal state. Think of a situation where you would like to call the same method multiple times and you need it to return different results. Here's where Mockito's `Answer` interface comes into play.

{% highlight %}
when(mock.someMethod(anyString())).thenAnswer(new Answer() {
     Object answer(InvocationOnMock invocation) {
         Object[] args = invocation.getArguments();
         Object mock = invocation.getMock();
         return "called with arguments: " + Arrays.toString(args);
     }
 });
{% endhighlight %}

##Verify passed parameters

##Verify number of invocations

##Capture method parameters

##Mock void methods