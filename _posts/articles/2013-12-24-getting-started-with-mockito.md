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
published: truelocalhost
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
In some cases it might be useful to keep internal state. Think of a situation where you would like to call the same method multiple times and you need it to return different results. Here's where Mockito's `Answer` interface comes into play. It provides us a way to create stateful mocks.

Let's look at the following example
{% highlight java %}
when(applicantQueue.getNextApplicant()).thenAnswer(new Answer<Applicant>() {
    private int counter;
    private String[] names = new String[] {"Mario", "Luigi", "Bowser"};

    @Override
    public Applicant answer(InvocationOnMock invocationOnMock) throws Throwable {
        return new Applicant(names[counter++]);
    }
});

assertEquals("Mario", appService.processNextApplicant().getApplicantName());
assertEquals("Luigi", appService.processNextApplicant().getApplicantName());
assertEquals("Bowser", appService.processNextApplicant().getApplicantName());
{% endhighlight %}

ApplicantQueue is an abstraction of a JMS queue. AppService has a mocked instance of `ApplicantQueue` and uses it in the `processNextApplicant` method to retrieve the next applicant. Using the `Answer` interface we can mock the behaviour of the queue and return deterministic results.

##Verify passed parameters

By now you should know how to mock complex objects and their behaviour in unit tests but it would be good to know how to verify the parameters passed to a mock. Mockito provides an `ArgumentCaptor` class which, as the name implies, can be used to capture arguments for further assertion.

##Verify number of invocations

##Capture method parameters

##Mock void methods