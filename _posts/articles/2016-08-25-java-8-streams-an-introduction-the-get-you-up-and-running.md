---
layout: post
title: "Java 8 Streams: an introduction the get you up and running"
excerpt:
modified: 2016-08-25 14:39:19 +0300
categories: articles
tags: [java, streams]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
aging: true
---

Even the simplest programs use some kind of collection of elements. They are an essential part in programming. Be it arrays, lists, dictionaries or maps, they're used to store data so it could be easily accessed. Depending on the use case, different data structures are chosen. Arrays are good for storing a sequence of elements. Key-value data can be stored in dictionaries or maps (some programming languages call them associative arrays). Up until Java 8, processing Collections in Java has been inconvenient.

I must admit that some operations are relatively simple to achieve but if you're faced with a task to filter and then group data, you're probably going to write several lines of code with multiple intermediary data containers. It would be better to declaratively define what operations you want to be applied to the collection. Think of SQL. You can select, filter and group data from a table declaratively. Java designers introduced the Streams API which allows to manipulate collections in a declarative and functional manner.

## Streams

Simple intro, declaratively (more readable as well) manipulate collections of data. Functional style together with lambda expressions. Pipe multiple operations together, think of cli piping -> composable
show before and after

## Other libraries

Guava, apache, lambdaj

traversable only once
parallel streams
functional -> map, reduce, filter, other stream usages
lazy
creating a stream from collections, streams from values, streams from arrays, streams from files, streams from functions
simple diagram of stream pipelining
intermediate operations vs terminal operations
data source -> intermediate operations -> terminal operation
truncating, limit()
skipping, skip()
finding
max min
numeric streams, specialized streams (intStream)
