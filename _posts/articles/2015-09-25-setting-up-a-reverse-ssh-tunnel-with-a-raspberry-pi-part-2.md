---
layout: post
title: "Setting up a reverse SSH tunnel with a Raspberry Pi part 2"
excerpt:
modified: 2015-09-25 20:52:14 +0300
categories: articles
tags: [raspberry pi, ssh, linux]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: false
---

Last time we got a Raspberry Pi set up and a VPS ready and accessible via SSH. If you have not read [part 1]({{site.url}}/articles/setting-up-a-reverse-ssh-tunnel-with-a-raspberry-pi/ "part 1 of setting up a reverse ssh tunnel with a raspberry pi") then I strongly encourage you to do so. In part 2 I promised to show an example use case on how to connect to a Windows Server over RDP via a reverse SSH tunnel.

##Testing SSH access

First, let's make sure we have SSH access to the Pi from our machine. I'm going to create an SSH session to the VPS and from there I'm going to create another SSH session to the Pi. This way I make sure that a reverse SSH tunnel is working as expected.

Connecting to the external server
{% highlight bash %}
$ ssh user@serverhostname
{% endhighlight %}

From there let's create another SSH session to our Pi which is inside the work network.
{% highlight bash %}
$ ssh pi@localhost -p 2222
{% endhighlight %}

Why connect to `localhost` on port `2222`? Recall from [part 1]({{site.url}}/articles/setting-up-a-reverse-ssh-tunnel-with-a-raspberry-pi/ "part 1 of setting up a reverse ssh tunnel with a raspberry pi") that we created a reverse SSH tunnel from the Pi to the server. We defined that all connections from our server to `localhost` on port `2222` should be sent to the Pi.

If you were able to connect to your Pi then the tunnel is working.

##Tunneling RDP traffic
