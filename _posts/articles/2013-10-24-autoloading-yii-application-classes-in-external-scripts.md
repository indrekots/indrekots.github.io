---
layout: post
title: "Autoloading Yii application classes in external scripts"
excerpt: ""
modified: 2013-10-24 20:43:34 +0300
categories: articles
tags: [yii, php, autoload, composer, cron]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
published: true
---

Sometimes I want to have access to all of my classes in a Yii app externally. For example, i would like to run an external PHP script with cron and I would like to use my application classes. To achieve that, I created a separate classloader file.

{% highlight php %}
<?php

// change the following paths if necessary
$yii=dirname(__FILE__).'/../vendor/yiisoft/yii/framework/yii.php';
$config=dirname(__FILE__).'/config/console.php';
$composer = dirname(__FILE__) . '/../vendor/autoload.php';

require_once($composer);
require_once($yii);
Yii::createConsoleApplication($config);
{% endhighlight %}

This creates a Yii console application and in addition autoloads all classes managed by Composer. 