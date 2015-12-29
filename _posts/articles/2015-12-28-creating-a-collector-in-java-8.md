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

##collector interface

##collect vs reduce
