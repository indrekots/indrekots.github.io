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

## Including other components

Finally, I added the ESP8266 controller to the car, connected ground and control pins with the motor driver and attached a LiPo battery. All the required components are in place. All that is left is to write the code that is going to transmit and receive commands over WiFi.

<figure>
	<a href="{{ site.url}}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/almost_ready.jpg" class="image-popup"><img src="{{ site.url}}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/almost_ready_thumb.jpg" alt="Amost ready"></a>
	<figcaption>Attached a LiPo battery and a SparkFun ESP8266 Thing board</figcaption>
</figure>

## Writing control logic

Debugging the SparkFun ESP8266 Thing is not as easy as a regular Arduino board. You cannot use the serial monitor provided by the Arduino IDE. [Luckily there is a simple solution for that](articles/using-a-serial-monitor-with-sparkfun-esp8266-thing/ "Using a serial monitor with SparkFun ESP8266 Thing").

I tested my initial ESP8266 implementations with **netcat**. For example, the following command sends a byte which corresponds to a *turn-right* command to the car. As you can see, the car has an IP address and a port it is listening on.

{% highlight bash %}
$ echo -e '\x03' | netcat 192.168.1.101 1111
{% endhighlight %}

Controlling the car with *netcat* is not fun. Therefore the next step was to implement a small Python program which would detect keyboard presses and send commands over the network to the car. I used the old Python program I wrote for [my previous project](https://github.com/indrekots/rc-car-controller) and [repurposed it](https://github.com/indrekots/esp8266-rc-car-controller/blob/master/wifi_controller_gui.py) so it could send commands over the network instead of over a serial connection.

## TCP vs UDP

During the first tests I discovered that the car was not very responsive. The car would continue driving when I had released all keys for example. Later I found that this was due to the fact that the communication was done over [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol "TCP Wikipedia page"). I'm guessing the TCP handshake process and acknowledgements introduce some overhead. After I had reengineered the car to use UDP everything worked flawlessly.

benefits - can control speed
