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

After doing some tests with a multimeter, you should have a pretty good understanding of the layout. The next step is to solder some wires to the circuit. I used 5 wires, 4 for each switch and 1 for ground. If you connect either of the four wires to ground, the corresponding switch should be closed and you should see a response from the car. Later I can plug the soldered wires to a breadboard.

##Build a circuit

When wires are soldered, the next step is to connect them to a breadboard and add some other electronic components there as well. I don't have a background in electrical engineering. Therefore I cannot draw you a schematic with the correct electrical symbols. But I can take a picture of the end result.

//TODO:insert image of circuit

Arduino pins 2, 3, 4 and 5 are optocouplers through a 220k ohm resistor. In the picture, pin 2 and 3 control the backwards and forwards switch respectively. Pin number 4 and 5 are responsible for turning left or right. So for example when pin 2 outputs 5 volts, the RC car starts to drive backwards.

##Putting it into code

My laptop is going to communicate with the Arduino using a serial connection over a USB cable. The goal is to send as little data as possible. I'm going to send a command only when a button is pressed or released. There's four different directions and two possible states, that means eight different commands in total.

###Python code
###Arduino code
