---
layout: post
title: "NTLM authentication fails on Linux when accessing QlikView Access Point"
excerpt: 
modified: 2014-10-09 21:01:12 +0300
categories: articles
tags: [qlikview, qv, access point, ntlm, linux, ubuntu, windows]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
published: false
---

I was unable to connect to QlikView Access Point from a Linux machine. My browser presented me with a username/password prompt and after entering the correct credentials no autentication was made it seemded. At first I thought the web server has been misconfigured. After testing I found that using a Windows computer the authentication worked flawlessy. Later I heard that clients were unable to connect with their OS X coputers.

The web server was configured to use NTLM authentication.

> In a Windows network, NT LAN Manager [NTLM](https://en.wikipedia.org/wiki/NT_LAN_Manager "NTLM Wikipedia page") is a suite of Microsoft security protocols that provides authentication, integrity, and confidentiality to users