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

## Why use the laptop as a transmitter?

It does not have to be a laptop. If I can make the RC car controllable programmatically, I could use other WiFi connected devices as well (e.g. smartphone). For the initial prototype I feel it is reasonable to test the connection with a laptop.

## ESP8266 chip

They come in different shapes and sizes. For my project I went with the [SparkFun ESP8266 Thing
](https://www.sparkfun.com/products/13231 "SparkFun ESP8266 Thing product page") because the pins are broken out to two parallel, breadboard-compatible rows and it comes with a lithium polymer battery which is useful since the RC car is not plugged in to a power source. The Thing can be even programmed using the Arduino IDE by installing the [ESP8266 Arduino addon](https://learn.sparkfun.com/tutorials/esp8266-thing-hookup-guide/installing-the-esp8266-arduino-addon "ESP8266 arduino addon").

## Getting started

As mentioned previously, in my [first project]({{ site.url }}/articles/controlling-an-rc-car-with-arduino/ "Control an RC car with an Arduino") I did not open up the car itself. Obviously, if I want to start using a ESP8266 chip, I'm going to have to place it inside the car. So off I went to find a screwdriver.

After opening the car I found that it uses the standard RX-2/TX-2 chipset used in many cheap RC toys. Additionally the car has two DC motors for moving and steering and LED head/taillights. After removing the original printed circuit board inside the car, I have room to place the ESP8266 chip in it.

<figure class="half">
  <a href="{{ site.url }}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/inside.jpg" class="image-popup">
	  <img src="{{ site.url }}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/inside.jpg" alt="Inside of the RC car">
  </a>
  <a href="/images/2016-07-11-how-to-control-an-rc-car-over-wifi/rx2tx2.jpg" class="image-popup">
	  <img src="/images/2016-07-11-how-to-control-an-rc-car-over-wifi/rx2tx2.jpg" alt="The chipset removed from the car">
  </a>
	<figcaption>A view of the inside of the car, RX-2/TX-2 chip removed</figcaption>
</figure>

## Controlling the motors

I can't just plug the power and ground wires to my ESP8266. Firstly the chip can output only 3.3V but the motors require more. Secondly, I need to control the rotation direction of the motors. For power, I'm going to use the original batteries that are mounted under the car. The control logic will be powered by the LiPo battery included with the SparkFun ESP8266 Thing. For controlling the motors I decided to use the [Pololu DRV8833 Dual Motor Driver Carrier](https://www.pololu.com/product/2130 "DRV8833 Dual Motor Driver Carrier product page"). As the name implies, it can handle two motors.

[insert image of batteries]

## Setting up the motor driver

Reading the manual from Pololu helped me understand how to use the motor driver. They have create schematics and descriptions for each pin.

<figure>
	<a href="{{ site.url}}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/wiring_diagram.png" class="image-popup"><img src="{{ site.url}}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/wiring_diagram.png" alt="Pololu DRV8833 wiring diagram"></a>
	<figcaption><a href="https://www.pololu.com/product/2130" title="Pololu DRV8833 wiring diagram">Pololu DRV8833 wiring diagram</a></figcaption>
</figure>

Before taking out my soldering iron, I tested the motor driver on a breadboard to make sure I understood everything correctly. At that point I had not yet implemented the code which would control the car. For the test I just turned the control pins high one by one to see if the motor started rotating. Then the next step was to connect the motor driver with the motors in the car using my limited soldering ability.

<figure class="half">
  <a href="{{ site.url }}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/breadboard_test.jpg" class="image-popup">
	  <img src="{{ site.url }}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/breadboard_test_thumb.jpg" alt="Testing the motor driver on a breadboard">
  </a>
  <a href="/images/2016-07-11-how-to-control-an-rc-car-over-wifi/soldered_driver.jpg" class="image-popup">
	  <img src="/images/2016-07-11-how-to-control-an-rc-car-over-wifi/soldered_driver_thumb.jpg" alt="Motor driver connected with the car's DC motors">
  </a>
	<figcaption>Testing out the motor driver on a breadboard and connecting it with the car</figcaption>
</figure>

intro, link to prev posts
esp8266
set up
tcp vs udp
benefits - can control speed
