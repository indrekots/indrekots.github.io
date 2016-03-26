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
[Optionals in Java](({{site_url}}/articles/optionals-in-java-8/ "What are Optionals?")) can help you to write a program without *null checks*. In addition they can make your code more concise and easier to read. Let's look at an example how chaining Optionals in a *Stream* can help you avoid unnecessary *null checks*.

The purpose of chaining optionals in a stream is to pick the first Optional which has a value and return it. You could have multiple service methods which return optionals. Passing them to a stream as method references allows you to process them lazily and return as soon as there's something to return.

The following utility method takes *Suppliers* as arguments and returns the first Optional which has a value.
{% highlight java %}
public static <T> Optional<T> first(Supplier<Optional<T>>... suppliers) {
    return Arrays.asList(suppliers).stream()
            .map(Supplier::get)
            .flatMap(o -> o.map(Stream::of).orElseGet(Stream::empty))
            .findFirst();
}
{% endhighlight %}

It is possible to use method references or lambda expressions.

{% highlight java %}
first(this::find1, this::find2, this::find3);
{% endhighlight %}

JDK8 does not provide a convenient API for converting a Stream out of an Optional. Apparently this is going to be [fixed in JDK9](https://bugs.openjdk.java.net/browse/JDK-8050820 "OpenJDK issue tracker") and the utility method could be written as

{% highlight java %}
public static <T> Optional<T> first(Supplier<Optional<T>>... suppliers) {
return Arrays.asList(suppliers).stream()
        .map(Supplier::get)
        .flatMap(Optional::stream)
        .findFirst();
{% endhighlight %}

Think about how to implement the same concept without using Optionals or lambda expressions. It would probably include multiple null checks. Using a more functional style can help you implement a more general solution which is hopefully easier to understand as well.
