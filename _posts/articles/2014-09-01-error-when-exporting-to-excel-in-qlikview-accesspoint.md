---
layout: post
title: "Error when exporting to Excel in QlikView AccessPoint"
excerpt: 
modified: 2014-09-01 21:15:12 +0300
categories: articles
tags: [excel, qv, qlikview]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
published: false
---

{% highlight xml %}
<result><message text="Empty response" /></result>
{% endhighlight %}

To fix it, make sure in QVS -> Folders the alternate temporary files folder path is set and it actually exists
In addition QVWS->Web make sure .XLSX is added to the mime types with content: application/vnd.ms-excel

//todo: add images
