---
layout: post
title: "Controlling an RC car with Arduino"
excerpt:
modified: 2016-01-23 18:12:45 +0200
categories: articles
tags: [arduino, python, rc car]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
---
Driving an RC car is fun. But I think it's much more fun to break things apart and see what else can be done with it. I decided to buy a cheap RC car and try to control it using a keyboard and an Arduino. If you have any experience in programming but have not touched electronics before, you're still able to follow along. It's not difficult at all.

The way I'm going to control an RC car is to open up its remote and plug an Arduino to it. This way i'm able to programmatically control the switches.

Things you're going to need:

* A cheap RC car (obviously)
* An Arduino board
* Soldering equipment
* Multimeter
* Some electronic components
  * 4 octocouplers
  * 4 220k ohm resistors
  * a breadboard
  * jumper wires

##Open the remote

When you have opened up the remote, you can see switches on the circuit board. The idea is that instead of pushing the switch you can simulate a switch being pressed with an Arduino. For that I'm going to use an optocoupler. They're integrated circuits which allow you to control devices that are on a different circuit. Inside an optocoupler is an LED and a light detector. When the LED is turned on the light detector closes a switch. This allows you to connect an Arduino to a different circuit and have them be electrically separated.

![Optocoupler schematic]({{ site.url }}/images/2016-01-23-controlling-an-rc-car-with-arduino/optocoupler_diagram.jpg "Optocoupler schematic")

Looking at the circuit, you need to determine which parts of the circuit are closed by the switch. Take out a multimeter, put it into diode mode and try to connect one end of the switch to ground. If you have the RC car turned on you should see a response from the car.

##Solder wires

After doing some tests with a multimeter, you should have a pretty good understanding of the circuit. The next step is to solder some wires on the circuit. I used 5 wires, 4 for each switch and 1 for ground. If you connect either of the four wires to ground, the switch should be closed and you should see a response from the car. Later I can plug the soldered wires to a breadboard.

##Build a circuit
##Arduino code
##Python code
