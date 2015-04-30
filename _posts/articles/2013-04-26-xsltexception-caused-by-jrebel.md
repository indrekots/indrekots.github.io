---
layout: post
title: "XsltException caused by JRebel"
excerpt: "XsltException caused by a seemingly missing stylesheet. Turns out JRebel was the culprit."
modified: 2013-04-18 18:26:17 +0300
categories: articles
tags: [java, jrebel, windows, maven, pom.xml]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
---

I stubled upon a problem where a Java web application complained about an XsltException which was caused because a stylesheet was not found. The stylesheet existed but for some reason it was not found. 