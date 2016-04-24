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
published: true
---

Since the beginning of multicore processors, the most effective way to increase the performance of an application is to fully utilize all processor cores. Running tasks which might take longer to return asynchronously (in a separate thread) is beneficial because the main thread can continue to do useful work while your result is being calculated. Using blocking calls can considerably decrease throughput as the program is wasting useful clock cycles on waiting for the call to return. Achieving greater throughput is also possible by splitting a long running task into smaller tasks and run them in parallel.

CompletableFuture is an implementation of the Future interface but with a modern twist. It takes advantage of functional operations to promote asynchronous/event-driven programming model. You should have some knowledge of threads in Java to be able to follow along.

## Future

To better understand CompletableFutures, let's have a look at Futures first. They model asynchronous computations and provide a reference to its result which might not yet be available. Methods are provided to get the result or check if the computation has finished. Calls to methods returning Futures can return immediately and the calling thread can continue to do useful work. All in all,  Futures are friendlier to use than lower-level threads.

Wrap the operation in a Callable and pass it to an Executor service
Code example
Describe code example

.get() and .get() with timeout

Downsides of future and why CompletableFuture

## CompletableFuture

Declarative features for combining results from multiple futures, dependencies between jobs.
CompletableFuture is to a plain Future what Stream is to a Collection.

Synchronous vs asynchronous calls.

code example
completing with exception/timeout

when to use parallel streams and completablefutures (Parallelism—via Streams or CompletableFutures? in book)

Pipelining async tasks
pipelining example code, thenApply(), thenCompose(), thenCombine(), thenAccept()
