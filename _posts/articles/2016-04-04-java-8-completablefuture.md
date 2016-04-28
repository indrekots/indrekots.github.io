---
layout: post
title: "Java 8: CompletableFuture"
excerpt:
modified: 2016-04-04 07:20:35 +0300
categories: articles
tags: [java, java 8, multithreading]
image:
  feature: 2016-04-04-java-8-completablefuture/cover.jpg
  credit: http://thebraincandypodcast.com/
  creditlink: http://thebraincandypodcast.com/episode-3-the-future/
comments: true
share: true
published: true
---

Since the beginning of multicore processors, the most effective way to increase the performance of an application is to fully utilize all processor cores. Running tasks which might take longer to return asynchronously (in a separate thread) is beneficial because the main thread can continue to do useful work while your result is being calculated. Using blocking calls can considerably decrease throughput as the program is wasting useful clock cycles on waiting for the call to return. Achieving greater throughput is also possible by splitting a long running task into smaller tasks and run them in parallel.

CompletableFuture is an implementation of the Future interface but with a modern twist. It takes advantage of functional operations to promote asynchronous/event-driven programming model. You should have some knowledge of threads in Java to be able to follow along.

## Future

To better understand CompletableFutures, let's have a look at Futures first. They model asynchronous computations and provide a reference to its result which might not yet be available. Methods are provided to get the result or check if the computation has finished. Calls to methods returning Futures can return immediately and the calling thread can continue to do useful work. All in all,  Futures are friendlier to use than lower-level threads.

To create a future you need to wrap an operation in a Callable and pass it to an ExecutorService.

{% highlight java %}
ExecutorService executorService = Executors.newSingleThreadExecutor();

Future<String> result = executorService.submit(() -> {
    delay(5000); //simulate network call
    return "{\"result\":\"success\"}";
});

IntStream.range(1, 10).forEach(i -> {
    delay(100);
    System.out.println("Doing useful work");
});
System.out.println(result.get());
{% endhighlight %}

In the previous example a Callable is passed to an ExecutorService. Since Java 8, instead of an anonymous class you can use a lambda expression to define the body of the Callable. The `delay` method is used to simulate a network call which delays the thread for 5 seconds. ExecutorService will return a Future which can be used later to retrieve the result. During the time the network call takes place, it is possible to do other useful work. Finally calling `get()` on the Future will get its result. If it's not available yet, the execution is blocked until the result becomes available. But what if the thread never finishes or takes too long to compute? The [*get* method](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Future.html "Future JavaDoc") can accept a timeout as an argument. You can also use the `isDone()` method to check if the task has completed.

But what if you'd like to take the result of an asynchronous task and immediately do other computations with it and then, say, combine the result with a result from another async task. This is where CompletableFutures come in handy because they allow to express dependencies between tasks declaratively.

## CompletableFuture

The name CompletableFuture comes from the fact that it is a Future which can be explicitly completed by calling the `complete()` method. Instead of using an ExecutorService to return a Future, completing a CompletableFuture can be done by just creating a thread.

As in the previous example with a Future, let's say we're making a network call to retrieve some data.

{% highlight java %}
public Future<String> getResultOverNetwork() {
    CompletableFuture<String> future = new CompletableFuture<>();
    new Thread(() -> {
        delay(5000); //simulate network call
        future.complete("{\"result\":\"success\"}");
    }).start();
    return future;
}
{% endhighlight %}

The computation (in this case a simulated network call) is done in a separate thread. A lambda expression is used to define the Runnable passed to the thread. Before returning a Future, the thread is started. The `getResultOverNetwork()` method returns almost instantaneously, leaving the calling thread free to to other useful work.

{% highlight java %}
Future<String> future = getResultOverNetwork();
IntStream.range(1, 10).forEach(i -> {
    delay(100);
    System.out.println("Doing useful work");
});
System.out.println(future.get());
{% endhighlight %}

Technically this example is almost the same as shown previously with a regular Future. The only difference is that instead of a plain old Thread, an ExecutorService was used. To be fair though, CompletableFuture class has some static methods which make the usage a little bit easier as can be seen in the following example.

{% highlight java %}
public Future<String> getResultOverNetwork() {
    return CompletableFuture.supplyAsync(() -> {
        delay(5000);
        return "{\"result\":\"success\"}";
    });
}
{% endhighlight %}

But still, what's so special about CompletableFutures?

### Pipelining tasks

Declarative features for combining results from multiple futures, dependencies between jobs.
CompletableFuture is to a plain Future what Stream is to a Collection.

Synchronous vs asynchronous calls.

code example
completing with exception/timeout

when to use parallel streams and completablefutures (Parallelism—via Streams or CompletableFutures? in book)

Pipelining async tasks
pipelining example code, thenApply(), thenCompose(), thenCombine(), thenAccept()
