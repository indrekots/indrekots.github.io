---
layout: post
title: "Service won't start when using the service utility"
excerpt: I debugged a service which didn't start when using the service utility but was working fine when calling the init script directly.
modified: 2015-12-16 21:20:56 +0200
categories: articles
tags: [upstart, system v, service, linux, ubuntu, mule, esb]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
---

Recently I learned something new and interesting. I was trying to run Mule on Ubuntu 14.04.

>Mule ESB is a lightweight Java-based enterprise service bus (ESB) and integration platform that allows developers to connect applications together quickly and easily, enabling them to exchange data.

To get the latest version I downloaded the binaries from Mule's website. Since I wanted it to be run as a service, I created the following init script (in `/etc/init.d/mule`).

{% highlight bash %}
#!/bin/bash

JAVA_HOME=/usr/lib/jvm/java-8-oracle
PATH=$PATH:$JAVA_HOME/bin

# Set Mule related environment
MULE_HOME=/opt/mule
PATH=$PATH:$MULE_HOME/bin
RUN_AS_USER=mule

# Export environment variables
export JAVA_HOME MULE_HOME RUN_AS_USER PATH

# Invoke Mule
$MULE_HOME/bin/mule $1
{% endhighlight %}
