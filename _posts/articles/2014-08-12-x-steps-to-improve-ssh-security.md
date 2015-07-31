---
layout: post
title: "X steps to improve SSH security"
excerpt: In this post you'll find some easy steps to secure your ssh server.
modified: 2014-08-12 07:53:49 +0300
categories: articles
tags: [ssh, linux, security, shell, secure shell]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
published: false
---

##Disable root login

Only give permission to unprivileged accounts to log in, later if needed they can escalate privileges via sudo
root does not need to have an ssh login, benefits-> one extra step of security, if everyone uses their own account, it is possible to audit who did what when.

PermitRootLogin no
AllowUsers <name>

##Use ssh keys

Generate a key pair with ssh-keygen, use ssh-copy-id to move pub key to server

ssh-keygen -t rsa -C <email>

ssh-copy-id -i <pub key> <user>@<host> 

##Disable password authentication

Passwords are bad but that's the best we got, ssh keys are better, use fail2ban if you want to use password login

##Run ssh on non standard port

Although this is security by obscurity, it still helps against most script kiddies, using nmap you can find which ports are open

##Run on protocol 2

Although this should be the default, it's worth checking that ssh-2 is used, check /etc/ssh/sshd_config
