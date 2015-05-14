---
layout: post
title: "Configuring Tomcat to accept Estonian ID card"
excerpt: "In order to accept Estonian ID card, Tomcat needs to be configured to accept client side certificates."
modified: 2013-09-20 18:17:21 +0300
categories: articles
tags: [apache, tomcat, esteid, client certificates, ssl, tls, java]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
published: true
---

In order to accept Estonian ID card, Tomcat needs to be configured to accept client side certificates. But before that, [SSL needs to be configured]({{site.url}}/articles/configure-ssl-on-apache-tomcat "Configure SSL on Apache Tomcat").

##Setting up client side authentication

First you need all SK's root certificates. 

> SK (Certification Centre, legal name AS Sertifitseerimiskeskus) is Estonia's primary and currently the only certification authority (CA), providing certificates for authentication and digital signing to national identity documents (ID-card, residence permit, Digi-ID, Mobiil-ID).

{% highlight bash %}
$ wget http://www.sk.ee/crls/esteid/esteid2007.crl
$ wget http://www.sk.ee/crls/juur/crl.crl
$ wget http://www.sk.ee/crls/eeccrca/eeccrca.crl
$ wget http://www.sk.ee/repository/crls/esteid2011.crl
{% endhighlight %}

Add these certificates to your keystore.

{% highlight bash %}
$JAVA_HOME/bin/keytool -importcert -file "/path/to/certs/EECCRCA.pem.cer" -alias eeccrca
$JAVA_HOME/bin/keytool -importcert -file "/path/to/certs/ESTEID-SK 2007.PEM.cer" -alias esteid2007
$JAVA_HOME/bin/keytool -importcert -file "/path/to/certs/ESTEID-SK 2011.pem.cer" -alias esteid2011
$JAVA_HOME/bin/keytool -importcert -file "/path/to/certs/JUUR-SK.PEM.cer" -alias juur
{% endhighlight %}

For each import, you will be asked for the keystore password and your confirmation on whether you trust this certificate or not.