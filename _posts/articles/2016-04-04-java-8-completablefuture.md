---
layout: post
title: "Java 8: CompletableFuture"
excerpt:
modified: 2016-04-04 07:20:35 +0300
categories: articles
tags: [java, java 8, multithreading]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
---

Increasing throughput using non-blocking operations
Pipelining and merging two or more asynchronous operations

since the advent of multicore processors, the most effective way to speed up your applications is to write software that’s able to fully exploit the multicore processors

This is part possible by splitting a larger task into smaller tasks and run them in parallel (reference to fork/join framework)
also parallel streams

Running tasks wich might take longer to respond asynchronously (e.g. getting some data from an external API). You don't want to block and waste clock cycles.

## Future

Since Java 5, models an asynchronous computation and provides a reference to its result which will be available when the computation. Allows the thread to continue and do other useful work. It's friendlier to use than lower-level threads.

Wrap the operation in a Callable and pass it to an Executor service
Code example
Describe code example

Downsides of future and why CompletableFuture
