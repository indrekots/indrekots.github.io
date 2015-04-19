---
layout: post
title: "Background jobs with PHP part 2"
modified: 2013-10-21 23:37:47 +0300
tags: [background, job, php, php-resque, phpredis, redis, resque, tutorial]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: true
published: false
---

Previously, in [part 1]({{site.url}}/background-jobs-with-php-part-1 "Part 1"), we took a look at how to implement a basic background job with PHP using [cron](http://en.wikipedia.org/wiki/Cron "Cron Wikipedia page"). In this post, as promised, we're going to implement a queue of jobs with [Resque](https://github.com/resque/resque "Resque Github page") and [Redis](http://redis.io/ "Redis homepage").