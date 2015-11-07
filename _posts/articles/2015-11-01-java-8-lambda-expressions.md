---
layout: post
title: "Java 8: Lambda expressions"
excerpt: In computer programming, an anonymous function (also function literal or lambda abstraction) is a function definition that is not bound to an identifier.
modified: 2015-11-01 18:17:27 +0200
categories: articles
tags: [java8, java, lambda]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
---

In computer programming, an anonymous function (also function literal or lambda abstraction) is a function definition that is not bound to an identifier ([Wikipedia](https://en.wikipedia.org/wiki/Anonymous_function "Anonymous function")). Many modern programming languages have had them for a long time. It's only Java that's a little late to the party. While being 20 years old, it has received lambda expressions relatively recently.

Before Java 8, Java developers could use anonymous inner classes to achieve something similar to lambda expressions. [In my last post]({{site.url}}/articles/java-8-behavior-parameterization "Java 8: Behavior parameterization") I showed that this is verbose and discourages the use of passing code as a parameter.

##Benefits

Technically lambdas do not help you do anything that you could not do prior to Java 8. Remember that anonymous inner classes can achieve the same result but with more boilerplate code. Therefore lambdas can improve your code by making it more clear on flexible.

**Before** (prior to Java 8):

{% highlight java %}
Collections.sort(books, new Comparator<Book>() {
    @Override
    public int compare(Book o1, Book o2) {
        return o1.getAuthor().compareTo(o2.getAuthor());
    }
});
{% endhighlight %}

**After** (using Java 8):

{% highlight java %}
books.sort((o1, o2) -> o1.getAuthor().compareTo(o2.getAuthor()));
{% endhighlight %}

In this example I sorted books by their author. That can be achieved with one line using a lambda expression. Technically this example shows another feature that Java 8 introduced as well - **default methods**. But that is not the topic of this post and I'm going to cover them in the future. In addition I'm going to show how to use **method references** and improve the line even more.

{% highlight java %}
books.sort(comparing(Book::getAuthor));
{% endhighlight %}



##syntax
##where to use, functional interface
##@FunctionalInterface
##functional interfaces in standard library
##execute around pattern, behavior parameterization
##type checking, type inference
