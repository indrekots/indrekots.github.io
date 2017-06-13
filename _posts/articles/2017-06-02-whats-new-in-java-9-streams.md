---
layout: post
title: "What's new in Java 9 Streams?"
excerpt:
modified: 2017-06-02 07:24:52 +0300
categories: articles
tags: [java, java 9, streams]
image:
  feature: 2017-06-02-whats-new-in-java-9-streams/cover.jpg
  credit: Caleb George
  creditlink: https://unsplash.com/@seemoris?photo=zZzKLzKP24o
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

// prefix
// parallel stream

## `dropWhile`

`dropWhile` is essentially the opposite of `takeWhile`. Instead of *taking* elements from the stream until the first element which does not match the predicate, `dropWhile` *drops* these elements and includes the remaining elements in the returned stream.

//longest contiguous prefix

The following is the same example we used previously with one noteworthy difference. `takeWhile` has been replaced with `dropWhile`.

{% highlight java %}
Stream.of(2, 4, 6, 8, 9, 10, 12)
    .dropWhile(n -> n % 2 == 0)
    .forEach(System.out::println);
// prints out:
// 9
// 10
// 12
{% endhighlight %}

In an ordered stream, `dropWhile` removes the longest contiguous sequence of elements that match the given predicate. In this example we're *dropping* even numbers. `2`, `4`, `6` and `8` are removed because applying the predicate on them returns `true`. `9` isn't an even number and is therefore included in the result. Even though `10` and `12` are even numbers, they're included in the result because they came after the first element which failed the predicate.

## Unordered streams

So far we've looked at how `takeWhile` and `dropWhile` behave with ordered streams. But what happens if the stream is unordered? According to the [docs](http://download.java.net/java/jdk9/docs/api/java/util/stream/Stream.html), if some of the elements in the stream match the predicate (but not all) then the operation is nondeterministic and an arbitrary subset of matching elements is returned or removed. Meaning that you'll get different results for each execution.

{% highlight java %}
Set<Integer> numbers = Set.of(2, 4, 6, 3, 8);
numbers.stream()
    .takeWhile(n -> n % 2 == 0)
    .forEach(System.out::println);
// prints out a different subset of matching elements every time
// an empty set is also a subset
{% endhighlight %}

You can expect a similar behavior if you replace `takeWhile` with `dropWhile`. It's free to drop any subset of matching elements. That includes the empty set.

## All elements match or none match

## Parallel streams

`takeWhile` and `dropWhile` are expensive operations on ordered parallel streams. Threads have to cooperate to find the longest contiguous sequence of matching elements in encounter order and this can significantly impact performance. Using a sequential stream may give you better results.

* Stream of nullable
* Stream iterate finite
