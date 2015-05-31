---
layout: post
title: "Setting up Redis replication"
excerpt: "The goal of this post is to install Redis on both of them and set up a master-slave replication configuration"
modified: 2013-10-24 20:43:34 +0300
categories: articles
tags: [redis, replication, ubuntu]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
published: true
---

I created 2 ubuntu server virtual machines. The goal of this post is to install Redis on both of them and set up a master-slave replication configuration. [Redis](http://redis.io/ "Redis homepage") is an open source, BSD licensed, advanced key-value cache and store.

To start, follow these steps:

* Install redis-server on both VMs
* To test if current setup is working, run `redis-cli`
* Then run ´SET keyname "fido"´ this will set a new key and its value
* To retrieve a keu value, run `GET keyname`, and the result should be "fido"  

##Configuring Redis

Now that we know Redis is working, let's continue configuring Redis. Configuration file is located at `/etc/redis/redis.conf`. 

To make `redis-server` connectable from the outside you need to open `redis.conf` and find the place where a bind  declaration is set. Remove or comment it out. If there is no bind statements, then all network interfaces will be listened for incoming connections. Save the file, close it and restart redis-server.

{% highlight bash %}
$ service redis-server restart
{% endhighlight %} 

##Making an instance of Redis a slave

Open the configuration file for Redis slave instance and add the following line

<pre>          
slaveof 192.168.10.101 6379 
</pre>

Default configuration should already contain a section for slaves which is commented out. Change the IP and port number to Redis master server IP and port and then restart the slave instance. 
 
{% highlight bash %}
$ service redis-server restart
{% endhighlight %} 
