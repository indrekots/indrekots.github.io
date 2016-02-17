---
layout: post
title: "The fork/join framework in Java"
excerpt:
modified: 2016-02-15 19:45:15 +0200
categories: articles
tags: [java, fork, join, concurrency]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
---

Introduced in Java 7, the fork/join framework is used to divide a parallelizable task into smaller ones and afterwards combine the intermediate results to get the end result. It is an implementation of the ExecutorService interface. In this post we're going to look at how to use the fork/join framework. We will create subtasks and submit them to the thread pool.

pseudocode

{% highlight java %}
if (task small enough) {
  do work sequentially
}
else {
  divide the task
  call the tasks recursively
}
{% endhighlight %}

parallel version of divide and conqure

which java versions support it (8, 7, 6?)

sample code

compare to executor service
