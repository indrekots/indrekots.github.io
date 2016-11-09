---
layout: post
title: "Running a Dropwizard application inside IntelliJ, Eclipse and NetBeans"
excerpt:
modified: 2016-11-09 16:33:22 +0200
categories: articles
tags: [intellij, eclipse, netbeans, dropwizard, java]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

[Dropwizard](http://www.dropwizard.io) is a framework for building [RESTful](https://www.sitepoint.com/what-does-restful-really-mean/) web services in Java. Similar to [Spring Boot](), Dropwizard applications are packaged into an executable *fat* JAR file. All external libraries your application depends on are present in a single file. The same artifact can be promoted from test to staging to production while being sure there’s no version changes in libraries used.

To execute a Dropwizard application, its `main` method needs to be called with the correct arguments. When you execute a JAR file from the command line, the JVM reads the entry point to your application from the `MANIFEST.MF` file. It contains name-value pairs and the key we're interested in is `Main-Class`. In a Dropwizard application, this is set to your `Applicaiton` class.

Equipped with this knowledge, it should be relatively easy to instruct your IDE to run a Dropwizard application during development.

## IntelliJ IDEA

1. Open *Run/Debug Configuration* menu by selecting Run -> Edit Configurations... from the menu.
2. Add a new configuration by pressing Alt+Insert or clicking the green plus sign in the top left corner.
3. From the dropdown menu, select *Application*.
4. Add a name to the configuration.
5. Enter the fully qualified name of your main class.
6. Enter `server config.yml` to the *Program arguments* field. These are the same arguments you would pass to your application if you were running it from the command line.

![IntelliJ IDEA run configuration menu]({{ site.url }}/images/2016-11-09-running-a-dropwizard-application-inside-intellij-eclipse-and-netbeans/idea_conf.png "Intellij IDEA Run configuration menu")
