---
layout: post
title: "What's new in Java 9 Streams"
excerpt:
modified: 2017-06-02 07:24:52 +0300
categories: articles
tags: [java, java 9, streams]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
aging: true
---

Arguably one of the most noteworthy new features introduced in Java 8 was the Streams API. It allowed developers to declaratively manipulate collections using functional style programming. Since the [release of Java 9](http://www.java9countdown.xyz/ "Countdown to Java 9 Release Date") is just around the corner, let's have a look at some of the improvements that have been made to the Streams API in Java 9.

## `takeWhile`

`takeWhile` is similar to `filter` in the sense that it expects a predicate and returns a new stream consisting only of the elements that match the given predicate. But there's a catch. In an ordered stream, `takeWhile` *takes* elements from the initial stream *while* the predicate holds `true`. Meaning that when an element is encountered that does not match the predicate, the rest of the stream is discarded. Let's have a look at the following example.

{% highlight java %}
Stream.of(2, 4, 6, 8, 9, 10, 12)
    .takeWhile(n -> n % 2 == 0)
    .forEach(System.out::println);

// prints out:
// 2
// 4
// 6
// 8
{% endhighlight %}

In this example, we're taking all even numbers from the initial stream until the first odd number is encountered. The stream returned by `takeWhile` contains `2`, `4`, `6`, `8`. It does not contain `9` since it did not match the predicate. Although `10` and `12` are even, they're not included in the returned stream as well because the stream operation was cut off when `9` was encountered.

But what happens if the stream is unordered? According to the [docs](http://download.java.net/java/jdk9/docs/api/java/util/stream/Stream.html#takeWhile-java.util.function.Predicate-), if some of the elements in the stream match the predicate (but not all) then the operation is nondeterministic and an arbitrary subset of matching elements is returned. Meaning that you get different results for each execution.

{% highlight java %}
Set<Integer> numbers = Set.of(2, 4, 6, 3, 8);
numbers.stream()
    .takeWhile(n -> n % 2 == 0)
    .forEach(System.out::println);
// prints out a different subset of matching elements every time
// an empty set is also a subset
{% endhighlight %}

## `dropWhile`

* Stream of nullable
* Stream iterate finite
