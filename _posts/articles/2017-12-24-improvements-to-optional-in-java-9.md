---
layout: post
title: "Improvements to Optional in Java 9"
excerpt:
modified: 2017-12-24 16:46:57 +0200
categories: articles
tags: [java, java 9, optional]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
aging: true
---
[Optionals were introduced to Java in version 8]({{site.url}}/articles/optionals-in-java-8/).
If used correctly, they can help you avoid unintentional `NullPointerException`s.
Essentially, an `Optional` is a container object which may or may not contain a non-null a value.
Combined with [lambda expressions]({{site.url}}/articles/java-8-lambda-expressions/), you can write concise code that avoids explicit `null` checks.

{% highlight java %}
Optional<Book> bookOptional = findBook("The War of the Worlds");
Book book = bookOptional.orElseGet(Book::defaultBook);
{% endhighlight %}

Java 9 introduced a couple of improvements to the [Optional](https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html) class.

## Stream from optional

`Optional::stream` allows you to transform an `Optional` to a `Stream`.
If a value is present, the returned `Stream` will contain that value.
Otherwise an empty `Stream` is returned.
You might be thinking how can that be useful?
We use Streams to manipulate collections of data.
How can an a stream that contains at most one element be useful?

In a stream processing pipeline we can transform optionals into streams and `flatmap` them into a single stream containing only the values that are present.
Let's look at the following example.

{% highlight java %}
List<Book> books = ...

List<Author> ghostwriters = books.stream()
  .map(Book::getGhostWriter)
  .flatMap(Optional::stream)
  .collect(toList());
{% endhighlight %}

Since every book doesn't have a [ghostwriter](https://en.wikipedia.org/wiki/Ghostwriter), `getGhostWriter` returns an `Optional` of `Author` (`Optional<Author>`).
For each book, `Optional::stream` creates a new stream and `flatmap` will ensure that all streams are merged together.
Essentially, we're getting rid of empty optionals.

You could do it in Java 8 as well with a [lambda expressions]({{site.url}}/articles/java-8-lambda-expressions/).

{% highlight java %}
List<Book> books = ...

List<Author> ghostwriters = books.stream()
  .map(Book::getGhostWriter)
  .flatMap(o -> o.map(Stream::of).orElseGet(Stream::empty))
  .collect(toList());
{% endhighlight %}

## Optional::or

[Optional::or](https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html#or-java.util.function.Supplier-) will either return the current Optional if a value is present.
Otherwise an Optional produced by the provided supplier function is returned.
To make things more clear, lets look at the following example.

{% highlight java %}

{% endhighlight %}

This is probably my favourite addition.

allows to chain optionals come in handy when you need to lazily evaluate the functions and stop when the first one returns something useful
Back in [Java 8 you had to go through some ceremony]({{site.url}}/articles/chaining-optionals-in-java-8/) to achieve something similar.
