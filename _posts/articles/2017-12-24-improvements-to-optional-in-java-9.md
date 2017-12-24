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

Java 9 brought some improvements to the [Optional](https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html) class.

## Stream from optional


## Optional::or

[Optional::or](https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html#or-java.util.function.Supplier-) will either return the current Optional if a value is present.
Otherwise an Optional produced by the provided supplier function is returned.
To make things more clear, lets look at an example.

{% highlight java %}

{% endhighlight %}

This is probably my favourite addition.

allows to chain optionals come in handy when you need to lazily evaluate the functions and stop when the first one returns something useful
Back in [Java 8 you had to go through some ceremony]({{site.url}}/articles/chaining-optionals-in-java-8/) to achieve something similar.
