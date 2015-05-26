---
layout: post
title: "Setting up Redis replication"
excerpt: "The goal of this post is to install Redis on both of them and set up a master-slave replication configuration"
modified: 2013-10-24 20:43:34 +0300
categories: articles
tags: [redis, replication]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
published: false
---

I created 2 ubuntu server virtual machines. The goal of this post is to install Redis on both of them and set up a master-slave replication configuration. [Redis](http://redis.io/ "Redis homepage") is an open source, BSD licensed, advanced key-value cache and store.

To start, follow these steps:
* Install redis-server on both VMs