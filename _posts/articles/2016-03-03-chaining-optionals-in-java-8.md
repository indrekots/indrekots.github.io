---
layout: post
title: "Chaining optionals in Java 8"
excerpt:
modified: 2016-03-03 07:16:38 +0200
categories: articles
tags: [java,java 8,optional]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
---
## Intro
What is optional
When to use it
Other libraries
Similarities from other languages

##Chaining optionals
motivation
feature missing from java 8
what's coming in jdk9

## Downsides
https://developer.atlassian.com/blog/2015/08/optional-broken/

## problems with null

If you have had any experience with Java, then you probably have seen the *NullPointerException* which is thrown when an application tries to use `null` in a case where an object is required. This can lead to superfluous if-statements checking to see if a reference is null or not.

Null references were implemented because they were the easiest method to implement the absence of a value. Tony Hoare, a British computer scientist, designer of the [ALGOL W programming language](https://en.wikipedia.org/wiki/ALGOL_W "Wikipedia page of ALGOL W") and the inventor of null references called it his [billion-dollar mistake](https://www.lucidchart.com/techblog/2015/08/31/the-worst-mistake-of-computer-science/ "Blog post about the worst mistake of computer science").

>I call it my billion-dollar mistake. It was the invention of the null reference in 1965. At that time, I was designing the first comprehensive type system for references in an object oriented language (ALGOL W). My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years. - Tony Hoare

## How to avoid NullPointerExceptions

The easiest answer is that you should never return a `null`. But that's easier said than done. What should I return when there is no value to return? Apparently null reference is not the best way to model an absence of a value. Instead you should return an object which represents an absence of a value. But wouldn't that lead to superfluous if-else statements checking if this object contains a value or not? Bear with me until I show how Optionals can solve this problem. But first, let's quickly go over how other languages have solved the same issue.

A lot of other languages have introduced a *Maybe* type - something that represents a sate where there might not be a value. For example, Scala has `scala.Option` and Standard ML uses `option`. Even if the language does not provide a concept of *Maybe*, there usually is a library that solves the problem. For instance, [Guava, the popular Java library by Google](https://github.com/google/guava "Guava Github page"), includes an [Optional class](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/base/Optional.html Guava Optional javadoc).

## Optional<T>

 
