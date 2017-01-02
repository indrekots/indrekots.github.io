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

In a barebones Spring Boot application you have an `application.properties` file in your classpath which is typically in the `resources` folder. Spring Boot reads properties from this file without you having to configure anything. But this is not externalized you say? And you're correct. As luck would have it...

application.properties in classpath
application.properties in outside of packaged jar
application.properties for each environment
spring devtools properties
application configuration in YAML format
environment variables

commandline arguments
