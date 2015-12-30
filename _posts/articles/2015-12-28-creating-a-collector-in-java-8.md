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

Streams help you process collections in a declarative manner. They support two types of operations: intermediate and terminal. While intermediate operations convert a stream to another stream, terminal operations consume the stream and return the final result. Java 8 [Stream interface](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html "Java 8 API Stream interface") defines a `collect` method which performs a mutable reduction operation on the elements of the stream. It accepts a `Collector` as a parameter which encapsulates the strategy that is used to compute the final result. In this post we'll have a look at how to create a collector from scratch.

##What is a collector?

A collector is a recipe for how to build a summary of the elements in a Stream. If you're familiar with Streams then you have probable seen the `toList()` collector.

{% highlight java %}
books.stream().
  filter(b -> b.getPageCount() > 100).
  collect(toList());
{% endhighlight %}

This collects all elements in a stream into a list. Among others, the `toList()` collector is part of the `Collectors` class which includes several other collectors as well.

##Existing collectors

To understand what collectors do and how they work, let's have a look at existing collectors in the Java API. In general, collectors can be divided into three broader categories:

* Reducing and summarizing stream elements to a single value
* Grouping stream elements
* Partitioning stream elements

###Collector returning a single value

An example of a collector which returns a single value is `counting()`. This counts the number of elements in a stream. Other notable collectors in this group include: `maxBy()`, `minBy()`, `summingInt()`.

###Collector which groups elements

A common database query might include a *group by* statement. It is possible to implement it with Java with an imperative style but it is cumbersome and very verbose. A `groupingBy()` collector can used to ease the pain of verbosity. The following is an example of a grouping collector.

{% highlight java %}
Map<String, List<Book>> booksByAuthor = books.stream().
  collect(groupingBy(Book::getAuthor));  
{% endhighlight %}

###Partitioning collector

Partitioning is technically a special case of grouping. A predicate (function which returns a boolean) is used to divide the stream into two groups. It returns a *Map* and its keys are booleans. So for example, to partition a stream of books into long and short ones you can use the following expression:

{% highlight java %}
Map<Boolean, List<Book>> booksByLength = books.stream().
  collect(Collectors.partitioningBy(b -> b.getPageCount() > 500));
{% endhighlight %}

##Collector interface

The [Collector interface](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collector.html "Collector interface javadoc page") defines a set of methods which are used during the reduction process. The following is the interface signature with the five methods it declares.

{% highlight java %}
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    Function<A, R> finisher();
    BinaryOperator<A> combiner();
    Set<Characteristics> characteristics();
}
{% endhighlight %}

* T is the type of items in the stream to be collected
* A is the type of the accumulator
* R is the type of the result returned by the collector

The `supplier()` must return a function that creates an empty accumulator. This will also represent the result of the collection process when applied on an empty stream. The job of the `accumulator()` is to return a function which performs the reduction operation. It accepts two arguments. First one being the mutable result container (accumulator) and the second one the stream element that should be folded into the result container. The `finisher()` returns a function which performs the final transformation from the intermediate result container to the final result of type R. Often times the accumulator already represents the final result, so the finisher can return *identity* function. When the stream is collected in parallel then the `combiner()` method is used to return a function which knows how to merge two accumulators. Finally, the `characteristics()` method returns an immutable set of `Characteristics` which define the behavior of the collector. This is used to check which kind of optimizations can be done during the reduction process. For example, if the set contains `CONCURRENT`, then the collection process can be performed in parallel.

##Building a custom collector

I gave a general overview of the *Collector* interface. This should be enough to start creating our own custom collector.

##collect vs reduce
