---
layout: post
title: "The four types of method references in Java 8"
excerpt:
modified: 2015-11-28 20:55:38 +0200
categories: articles
tags: [java]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
---

Method references let you reuse existing method definitions and pass them just like lambdas. They can be useful in making the code more readable. Instead of writing a full lambda expression, it is possible to point to an existing method.

Java 8 includes four types of method references:

* A method reference to a static method
* A method reference to an instance method of an arbitrary type
* A method reference to an instance method of an existing object
* Constructor reference

Let's go over each type.

##A method reference to a static method

This is probably the easiest to explain with the following example:

{% highlight java %}
Class::staticMethod
{% endhighlight %}

##A method reference to an instance method of an arbitrary type

The idea with this type of method reference is that you’re referring to a method of an object that will be supplied as one of the parameters of the lambda.

For example
{% highlight java %}
(String s) -> s.toUpperCase()
{% endhighlight %}

can be written as

{% highlight java %}
String::toUpperCase
{% endhighlight %}

##A method reference to an instance method of an existing object

With this method reference type you can refer to a method of an existing object. For example you might have an instance variable `books` which is of type `List<Book>`. Instead of referring to the size of the list with a full lambda expression

{% highlight java %}
() -> books.size()
{% endhighlight %}

you could use a method reference

{% highlight java %}
books::size
{% endhighlight %}
