---
layout: post
title: "How to control an RC car over WiFi"
excerpt:
modified: 2016-07-11 14:48:29 +0300
categories: articles
tags: [wifi, arduino, esp8266, IoT, hardware, electronics, python]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

Some time ago I got started with electronics and hardware hacking. For my first project I decided to [control a cheap RC car with an Arduino]({{ site.url }}/articles/controlling-an-rc-car-with-arduino/ "Control an RC car with an Arduino"). This was done by connecting my Arduino to the car's controller and sending commands from my laptop to the Arduino over a serial connection. The car would still receive the signal over 27MHz. It was a relatively easy project because I did not have to open up the car itself. The downside is that every time I want to take the car for a spin, I have to connect my laptop to my Arduino and then connect that to the controller.

## Ditching the controller

To make the setup easier, I decided not to use the original transmitter. My plan was not to have any extra wires dangling from my laptop and use the laptop itself as a transmitter. That left me with 2 possibilities - Bluetooth or WiFi. I decided to choose the latter since I found that there exists a thing called ESP8266 and I wanted to try it out. This is a low-cost WiFi chip with a full TCP/IP stack and a microcontroller capability.

## Why use the laptop as a transmitter

It does not have to be a laptop. If I can make the RC car controllable programmatically, I could use other devices as well (e.g. smartphone). For the initial prototype I feel it is reasonable to test the connection with a laptop.

## ESP8266 chip

They come in different shapes and forms. For my project I went with the [SparkFun ESP8266 Thing
](https://www.sparkfun.com/products/13231 "SparkFun ESP8266 thing product page") because the pins are broken out to two parallel, breadboard-compatible rows and it comes with a lithium polymer battery which is useful since the RC car is not plugged in to a power source. The Thing can be even programmed using the Arduino IDE by installing the [ESP8266 Arduino addon](https://learn.sparkfun.com/tutorials/esp8266-thing-hookup-guide/installing-the-esp8266-arduino-addon "ESP8266 arduino addon").

## Getting started

As mentioned previously, in my [first project]({{ site.url }}/articles/controlling-an-rc-car-with-arduino/ "Control an RC car with an Arduino") I did not open up the car itself. So obviously, if I want to start using a ESP8266 chip, I'm going to have to place it inside the car. So off I went to find a screwdriver.

After opening the car I found that it uses the standard RX-2/TX-2 chipset used in many cheap RC toys. Additionally the car has two DC motors for moving and steering.

<figure class="half">
  <a href="{{ site.url }}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/inside.jpg" class="image-popup">
	  <img src="{{ site.url }}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/inside.jpg" alt="Inside of the RC car">
  </a>
  <a href="/images/2016-07-11-how-to-control-an-rc-car-over-wifi/rx2tx2.jpg" class="image-popup">
	  <img src="/images/2016-07-11-how-to-control-an-rc-car-over-wifi/rx2tx2.jpg" alt="The chipset removed from the car">
  </a>
	<figcaption>A view of the inside of the car, RX-2/TX-2 chip removed</figcaption>
</figure>

intro, link to prev posts
esp8266
set up
tcp vs udp
benefits - can control speed
