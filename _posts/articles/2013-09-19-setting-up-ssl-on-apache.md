---
layout: post
title: "Setting up SSL on Apache"
excerpt: "This article explains how to create an SSL certificate, get it signed and configure Apache web server"
modified: 2013-09-19 17:24:19 +0300
categories: articles
tags: [apache, ssl, tls, pki, sni]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
published: false
---

**Disclaimer:** these instructions were tested on Ubuntu 13.04 and Apache 2.2.22.

Setting up SSL on the Apache web server consists of multiple steps. Firstly, a key pair must be generated. Based on that, a certificate signing request (CSR) can be made. And lastly, after a valid CA has issued an SSL certificate, Apache web server needs to be configured.