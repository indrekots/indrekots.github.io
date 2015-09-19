---
layout: post
title: "Setting up a reverse SSH proxy with a Raspberry PI"
excerpt:
modified: 2015-09-18 22:41:28 +0300
categories: articles
tags: [ssh, tunneling, raspberry pi, vpn, linux]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
---

This is kinda cool. Let's say you need to access your work computer from your home. You have a small office with no dedicated IT support. Probably you don't have a VPN which you could use to connect to your work network. But you have a spare Raspberry Pi lying around. You could put that into use and create a connection to your work machine by creating a reverse SSH proxy.

What you're going to need:
* a working Raspberry Pi
* physical access to your work network
* a server outside of your work network

##Setting up an outside server

You're going to need an outside server for your Pi to connect into. If you already have a box with a static IP and SSH running then you're good to go. If not, then you could create a cheap VPS at [DigitalOcean](https://www.digitalocean.com/?refcode=0b1905c10185 "link to DigitalOcean") for example.

##Setting up the Pi
