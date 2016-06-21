---
layout: post
title: "Design patterns: Observer pattern implementation in Java 8"
excerpt:
modified: 2016-06-21 14:36:56 +0300
categories: articles
tags: [design pattern, java, java8]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
---

A software design patterns is a solution to a commonly occurring problem. It is not a ready made piece of code but rather think of it as a best practice. Implementation details can vary from language to language so don't be alarmed when you see an implementation which differs from the one you're used to.

**Observer pattern**, described in [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612 "Design Patterns: Elements of Reusable Object-Oriented Software"), is a design pattern with an intent to define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

It is easier to think of it in terms of publishing and subscribing. Subscribers (also known as observers) subscribe to the publisher (also known as the subject) such that when the state of the publisher changes, it notifies all of its subscribers.

## Example domain

I'm going to use a video-sharing website (e.g. Youtube) as an example in this post. Users can *publish* new videos and *subscribe* to other users to get notified when a new video is available.

What is it?
What does it solve?
When to use it?
How to implement it
How to take advantage of Java 8
Where is it used?
JDK Observable
