---
layout: post
title: "How to configure VirtualBox VM with 2 network adapters"
excerpt: "This post describes how to create a VM which has a connection to the internet and also to other VMs in the virtual network."
modified: 2013-10-24 20:43:34 +0300
categories: articles
tags: [virtualbox, virtualization, vm, network]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
published: false
---

This post describes how to create a VM which has a connection to the internet and also to other VMs in the virtual network. The easiest way to achieve it would be to set the VMs to use a bridged network configuration. In some cases, it is not possible, because of the host's network setup. I guess this has something to do with the dhcp configuration. A bit of background. I have an Ubuntu installation as the host and 2 Ubuntu Server virtualmachines.
