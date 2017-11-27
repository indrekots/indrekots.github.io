---
layout: post
title: "Using environment variables with Spring Boot"
excerpt:
modified: 2017-11-27 07:17:09 +0200
categories: articles
tags: [spring, spring boot, java]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
aging: true
---

Spring Boot applications [can be easily configured via properties files]({{site.url}}/articles/externalized-configuration-in-spring-boot/ "Externalized configuration in Spring Boot").
But sometimes you need more flexibility in terms of how the configuration is provided to the application.
A popular approach next to configuration files is to use environment variables.
This post will cover how you can use OS environment variables to pass configuration values to a Spring Boot applicaiton.

## Injecting environment variables

You might already be familiar with the `@Value` annotation.
It can be used to inject property values to beans.
In addition to looking values from property files, the `@Value` annotation also looks for matching values from the environment.
So when you have an environment property called `DB_URL`, you can inject it to a field with `@Value("${DB_URL}")`.

{% highlight java %}
@Value("${DB_URL}")
private String dbUrl;
{% endhighlight %}
