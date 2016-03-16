---
layout: post
title: "Using a serial monitor with SparkFun ESP8266 Thing on Linux"
excerpt:
modified: 2016-03-15 07:19:11 +0200
categories: articles
tags: [sparkfun, esp8266, IoT, linux, arduino, SoC, gtkterm]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
---

[ESP8266](http://www.esp8266.com/ "esp8266 community") is a low-cost system on a chip with integrated TCP/IP stack and WIFI on board. The SparkFun ESP8266 Thing is a breakout and a development board for for the ESP8266 chip that makes it easier to use and provides some additional features.

If you have experience programming Arduino boards, then most likely you have used the serial monitor provided by the Arduino IDE to debug your code. It is possible to use the Arduino IDE to program ESP8266 chip as well provided that you install the required add-on. Unfortunately using the serial monitor provided by the IDE is not going to work with the SparkFun Thing.

//explanation form website

To use a serial monitor with SparkFun ESP8266 Thing on Linux you need to find an alternative serial monitor. I installed [gtkterm](https://fedorahosted.org/gtkterm/ "gtkterm website") and it can do the job pretty well.

//screenshots showing how to set up
