---
layout: post
title: "Externalized configuration in Spring Boot"
excerpt:
modified: 2016-12-31 16:19:06 +0200
categories: articles
tags: [spring, spring boot, configuration, java]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

Externalized configuration is essential if you wish to deploy the same deployable unit to different environments without modifying the source code. I'm certain you have aspects in your program that should change based on the execution environment. For starters, common parameters that need adjusting are the database connection strings or some cache specific values. Spring Boot provides several easy methods of externalizing configuration.

## Properties file

In a barebones Spring Boot application you have an `application.properties` file in your classpath which is typically in the `resources` folder. Spring Boot reads properties from this file without you having to configure anything. Application code can access these values using Spring's `Environment` or have them be injected into beans via the `@Value` annotation. This works great in development but when you start to deploy your application to other environments (e.g. test, production) you'll soon find out that you need to have different property values per environment. How can I do that with Spring Boot? I’m glad you asked, because that’s what we’ll look at next.

## Profile-specific properties file

In addition to the default `application.properties` file we talked about, you can create profile-specific properties files. [Profiles in Spring](http://docs.spring.io/autorepo/docs/spring-boot/current/reference/html/boot-features-profiles.html "Profiles in Spring Boot") allow you to configure certain parts of your application to suit the needs of different environments. For example, your database connection string might be different in your `test` environment compared to your `dev` settings. To create a profile-specific properties file you need to follow the following naming convention `application-{profile}.properties`.

Let's look at a trivial example. You have an `application.porperties` file with the following content.

{% highlight java %}
name=My Spring application
greeting=Hello
{% endhighlight %}

Alongside you also have an `application-test.properties` with one property defined.

{% highlight java %}
name=My Spring application in test
{% endhighlight %}

When you run your Spring Boot application, by default, property values from `application.properties` will be picked up. But when you start Spring Boot with `-Dspring.profiles.active=test`, you instruct Spring to look for properties form `application-test.properties`. I intentionally did not define a value for `greeting` in `application-test.properties`. When running with the `test` profile, Spring will first look into `application-test.properties` and for values which are not found there, it will use `applicaiton.properties`. Essentially profile-specific properties override non-specific ones.


The naive approach would be to pass the JVM an argument pointing to your external configuration file (e.g. `-Dconfig.location=external.properties`) and inside your application code read the properties from the specified file. This definitely works but Spring Boot offers a much simpler solution. Place your `application.properties` outside your packaged jar file. Spring Boot will look in the same directory for properties and the values can be accessed using Spring's `Environment` or injected into beans via the `@Value` annotation.




application.properties in classpath
application.properties in outside of packaged jar
application.properties for each environment
spring devtools properties
application configuration in YAML format
environment variables

commandline arguments
