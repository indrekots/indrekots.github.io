---
layout: post
title: "Creating a collector in Java 8"
excerpt:
modified: 2015-12-28 19:28:07 +0200
categories: articles
tags: [java, java 8, stream, collector]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
---

Streams help you process collections in a declarative manner. They support two types of operations: intermediate and terminal. While intermediate operations convert a stream to another stream then terminal operations consume the stream and return the final result. Java 8 [Stream interface](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html "Java 8 API Stream interface") defines a `collect` method which performs a mutable reduction operation on the elements of the stream. It accepts a `Collector` as a parameter. It encapsulates the strategy that is used to compute the final result.
