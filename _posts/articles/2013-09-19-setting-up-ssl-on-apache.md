---
layout: post
title: "Setting up SSL on Apache"
excerpt: "This article explains how to create an SSL certificate, get it signed and configure Apache web server"
modified: 2013-09-19 17:24:19 +0300
categories: articles
tags: [apache, ssl, tls, pki, sni, openssl]
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

##1. Generate public and private key

Generate keys using openSSL

{% highlight bash %}
$ openssl genrsa -out server.key 1024
{% endhighlight %}

##2. Generate CSR

CSR stands for Certificate Signing Request.

{% highlight bash %}
$ openssl req -new -key server.key -out server.csr
{% endhighlight %}

You are about to be asked to enter information that will be incorporated into your certificate request. What you are about to enter is what is called a Distinguished Name or a DN. There are quite a few fields but you can leave some blank. For some fields there will be a default value, If you enter '.', the field will be left blank.

The following are the questions being asked and sample answers.

<pre>
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
Please enter the following 'extra' attributes to be sent with your certificate request

A challenge password []:
An optional company name []:
</pre>

**The Common Name should be your server's DNS name.**
