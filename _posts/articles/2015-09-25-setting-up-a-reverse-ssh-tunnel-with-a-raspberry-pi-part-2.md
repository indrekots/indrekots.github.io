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

For the sake of the example I'm going to assume the IP of the Windows Server at the office network is `192.168.1.10` and the server outside of the network can be accessed via `myserver.com`. We're going to need to create two SSH tunnels, forwarding traffic from your machine to `myserver.com` and from there to the Pi.

###External server -> Raspberry Pi -> Windows Server

First of all, I'm going to create a tunnel to forward RDP traffic from `myserver.com` to the Windows Server.

{% highlight bash %}
$ ssh -L 3000:192.168.1.10:3389 pi@localhost -p 2222
{% endhighlight %}

This forwards all connections made to `localhost` on port `3000` to the Windows Server (`192.168.1.10`) using port `3389`. `-L` flag specifies that the given port on the local (client) host is to be forwarded to the given host and port on the remote side. It is achieved by creating an SSH tunnel to the Pi via a reverse SSH tunnel we set up previously. Note that `pi@localhost -p 2222` in the command is the same used previously to test the SSH tunnel.

###User machine -> External server

From your machine you should make an SSH tunnel forwarding all connections from `localhost` port `5000` (just a random port I chose) to the external server.

{% highlight bash %}
$ ssh -L 5000:localhost:3000 user@myserver.com
{% endhighlight %}

Now when you connect your RDP client to `localhost:5000`, this connection is forwarded to `myserver.com` and from there a new connection is made to `localhost:3000`. Looking at the preceding step, you can see that this will be in turn forwarded the Windows Server.
