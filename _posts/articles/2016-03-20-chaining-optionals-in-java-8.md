---
layout: post
title: "Chaining Optionals in Java 8"
excerpt:
modified: 2016-03-20 19:50:50 +0200
categories: articles
tags: [java, java 8, optional, lambda, functional programming]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
---

In one of my [previous posts]({{site_url}}/articles/optionals-in-java-8/) I described what are Optionals and how to use them in Java 8. In this post I'm going to go over how to chain optionals so that the first one that's present is returned. If none are present, an empty optional is returned. This method could be used in a use case where you need to pick a value from a list of sources and always pick the first one where the value is available.
