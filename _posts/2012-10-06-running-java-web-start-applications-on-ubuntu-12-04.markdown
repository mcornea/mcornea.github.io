---
layout: post
title: Running Java Web Start applications on Ubuntu 12.04
date: '2012-10-06 19:41:33 +0000'
categories:
- Linux
permalink: running-java-web-start-applications-on-ubuntu-12-04
---
In todays post I will present how to install the packages required to run Java web start applications ( .jnlp files) on Ubuntu 12.04.
Most of the KVM(keyboard, video and mouse) devices and several networking management software like Brocade Network Advisor use this kind of apps for remote access.

IcedTea is a web browser plugin running Java applets and Web Start apps. The icedtea-netx package automatically installs all the required Java Openjdk packages.

{% highlight bash %}
aptitude install icedtea-netx icedtea-netx-common icedtea6-plugin
{% endhighlight %} 

