---
layout: post
title: "Pattern Matching for instanceOf in Java 14"
excerpt: "Pattern matching capabilities added to the instanceOf operator in Java 14 is going to enhance the programming language."
modified: 2020-03-28 18:56:18 +0300
categories: articles
tags: [java, instanceOf]
image:
  path: /images/2020-03-27-java-14-instanceof/cover.jpg
  thumbnail: /images/2020-03-27-java-14-instanceof/cover_thumb.jpg
  caption: "Photo by [Omar Flores](https://unsplash.com/photos/lQT_bOWtysE)"
comments: true
share: true
published: true
aging: false
---

The `instanceOf` operator in Java is used to test whether an object is of a specific type.
A common usage pattern is to first check, then cast and finally use the extracted variable.
In this post we're going to look at the pattern matching capabilities added to the `instanceOf` operator in Java 14 and how it's going to enhance the programming language.

## `instanceOf` before Java 14

A typical usage for `instanceOf` is the _check-then-cast_ pattern.

```java
if (obj instanceof String) {
    var s = (String) obj;
    System.out.println(s.toUpperCase());
}
```

First we check whether `obj` is a `String` and then we cast `obj` into a `String`.
These steps go together so often that one might label this as boilerplate.
Next, we're going to look at how pattern matching tries to provide a better solution.

## Pattern Matching in Java 14

Starting from java 14, we can use `instanceOf` as follows.

```java
if (obj instanceof String s) {
    System.out.println(s.toUpperCase());
}
```

Casting is done at the same time as the `instanceOf` check.
To be technically correct, [JEP 305](https://openjdk.java.net/jeps/305 "JEP 305: Pattern Matching for instanceof") describes this as a _type test pattern_.

> A pattern is a combination of (1) a predicate that can be applied to a target, and (2) a set of binding variables that are extracted from the target only if the predicate successfully applies to it.
A **type test pattern** consists of a predicate that specifies a type, along with a single binding variable.

`instanceOf` operator matches `obj` to the type test pattern.
If the test passes, casting is done automatically and variable `s` is of type `String` in the `true` block of the `if` statement.
In more complicated conditional expressions, the scope of `s` grows as well.

```java
if (obj instanceof String s && s.length() > 5) {
    System.out.println(s);
}
```

## Summary

Pattern matching for `instanceOf`, [along with other features introduced in Java 14](https://openjdk.java.net/projects/jdk/14/),
make the language nicer to use and show that the decision to [move to a 6 month release cadence](https://www.infoq.com/news/2017/09/Java6Month/ "Java to Move to 6-Monthly Release Cadence") back in 2017 helps the language to evolve faster.

Keep in mind, pattern matching for `instanceOf` in Java 14 is a [preview feature](https://openjdk.java.net/jeps/12 "JEP 12: Preview Language and VM Features").
The implementation could change and evolve in upcoming releases.
To unlock preview features, use the `--enable-preview` command line flag for `javac` and `java`.
