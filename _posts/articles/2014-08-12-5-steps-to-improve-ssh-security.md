---
layout: post
title: "5 steps to improve SSH security"
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

SSH is something we (developers, sysadmins) use daily to connect to remote servers. Here are a few tips to keep your SSH connection secure.

##Disable root login

Allowing `root` to log in is not the safest idea. If you have a public facing server then there's a good change bots are attempting to log in. They do so by using known usernames that probably exists. One of them is `root`. Using uncommon usernames introduces an additional parameter the bots have to guess.

Every user should have their own account. Later, if needed, they can use `sudo` to gain more privileges. This adds one extra layer of security. If a standard user's account is compromised, the attacker still needs a sudo password to do more damage. Additionally, mandating each user to use their own account facilitates auditability. 

In `/etc/ssh/sshd_config`

{% highlight xml %}
PermitRootLogin no
AllowUsers <username>
{% endhighlight %}

##Disable password authentication

The reason here is simple. People come up with bad passwords. Besides being too short and never truly random, same passwords are likely being used in multiple places. If you still must use password authentication, use something like [Fail2ban](http://www.fail2ban.org/ Fail2ban webpage) to monitor for malicious signs.

Disable password login in `/etc/ssh/sshd_config`
{% highlight php %}
PasswordAuthentication no
{% endhighlight %}

##Use ssh keys

people come up with bad passwords and tend to reuse them
Generate a key pair with ssh-keygen, use ssh-copy-id to move pub key to server

ssh-keygen -t rsa -C <email>

ssh-copy-id -i <pub key> <user>@<host> 

##Run ssh on non standard port

Although this is security by obscurity, it still helps against most script kiddies, using nmap you can find which ports are open

##Run on protocol 2

Although this should be the default, it's worth checking that ssh-2 is used, check /etc/ssh/sshd_config
