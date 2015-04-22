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
published: true
---

Previously, in [part 1]({{site.url}}/background-jobs-with-php-part-1 "Part 1"), we took a look at how to implement a basic background job with PHP using [cron](http://en.wikipedia.org/wiki/Cron "Cron Wikipedia page"). In this post, as promised, we're going to implement a queue of jobs with [Resque](https://github.com/resque/resque "Resque Github page") and [Redis](http://redis.io/ "Redis homepage").

Redis is an open source, BSD licensed, advanced key-value store. It is often referred to as a data structure server since keys can contain strings, hashes, lists, sets and sorted sets.

###Dependencies

To follow this tutorial, the following software requirements must be met.

* Redis - key-value store
* Predis - library for accessing Redis key-value store
* php-resque - PHP port of Resque

###Installation

Redis can be downloaded at [http://redis.io/download](http://redis.io/download "Redis download link"). If you're running some flavour of Linux, then you might find Redis in it's repositories as well. Predis can be acquired at the project's [Github page](https://github.com/nrk/predis "Predis Github page"). It is also available on [Packagist](https://packagist.org/packages/predis/predis "Predis packagist page") for an easy installation using [Composer](http://getcomposer.org/ "Composer homepage"). Finally, php-resque can be found on [Github](https://github.com/chrisboulton/php-resque "php-resque Github page") as well. Composer users can find it on [Packagist](https://packagist.org/packages/chrisboulton/php-resque "php-resque on Packagist").

###General overview

We are going to create background workers who are going to process jobs. Jobs are pushed and held in a queue. Workers pull jobs from the queue and execute them. In essence, there are 4 parts to the system:

* Queues - jobs are held here in order
* Jobs - a unit of work to be processed
* Pushers - a process that  pushes  jobs into queues. This process can be anything, a request from the web server or even a worker that creates additional jobs from its previous input.

###Job

Job is an order for the worker to execute a certain task. To create a job with php-resque, you need to create a new class which implements a `perform()`  method. I would create a new interface which defines the `perform()` method and let concrete job classes implement the Job interface.

~~~ php
/**
 * Defines a php-resque job
 */
interface Job
{
 
  /**
   * Executes the job
   */
  public function perform();
 
}
~~~ 

~~~php
class HelloJob implements Job
{
  public function perform()
  {
    echo "Hello world";
  }
}
~~~

Of course in a real application, this job would do something more important and probably be more complex as well. Now that we have a simple job, let's create workers, who will execute the job.

###Workers

To create a new worker, you need to provide some environment variables. Firstly, the QUEUE variable defines which queue the worker is going to poll.