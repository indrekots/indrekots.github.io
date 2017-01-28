---
layout: post
title: "Type safe configuration in Spring Boot"
excerpt:
modified: 2017-01-14 13:13:48 +0200
categories: articles
tags: [java, spring, spring boot, configuration]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
aging: true
---

Spring Boot makes is very easy to read values from an external properties/yml file. Without any configuration, you are able to inject values to instance fields using the `@Value` annotation or use Spring's `Environment` abstraction. But both of these solutions require you to specify the property value as a string. Spring Boot also provides a type-safe way to handle configuration.

validation, strongly typed

@ConfigurationProperties("foo")
class

inject bean to other beans
