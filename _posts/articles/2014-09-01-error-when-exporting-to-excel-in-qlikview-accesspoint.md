---
layout: post
title: "Error when exporting to Excel in QlikView AccessPoint"
excerpt: 
modified: 2014-09-01 21:15:12 +0300
categories: articles
tags: [excel, qv, qlikview, xls]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
published: false
---
A client of mine was unable to export data from QlikView Access Point to Excel. Instead of a spredsheet file, the web server returned the following ambiguous result.

{% highlight xml %}
<result><message text="Empty response" /></result>
{% endhighlight %}

I was able to fix it by setting the alternate temporary files folder path in QlikView Management Console. Go to System -> Setup. Select your QlikView Server and then the Folders tab. At the bottom of the page you should see the following form.

![alternate temp folder form]({{ site.url }}/images/alternate_temp_folder.png)

Make sure the folder actually exists and is writable.


In addition QVWS->Web make sure .XLSX is added to the mime types with content: application/vnd.ms-excel

//todo: add images
