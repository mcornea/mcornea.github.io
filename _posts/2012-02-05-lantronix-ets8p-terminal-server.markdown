---
layout: post
title: " Lantronix ETS8P Terminal Server"
date: '2012-02-05 20:14:04 +0000'
categories:
- Linux
- terminal-server
permalink: lantronix-ets8p-terminal-server
---
I recently got a terminal server for my lab because as the number of devices grows it was difficult to manage them all by USB console adapters. So i found a cheap and quite old terminal server - Lantronix ETS8P - a terminal server with 8 serial RJ45 ports and 1 x 10Mbps ethernet interface. It's actually that old that it doesn't even support SSH, you can remote access it just by telnet. 

___

Anyways, I was very excited when I got it and I got right away a console cable on my laptop and connected to it - &nbsp;I was thinking I will have it configured in a couple of minutes but in the end it took me a couple of more minutes to have it ready.

I configured an IP address for it, prepared 8 rollover cables for connecting to Cisco devices, connected my devices and after that there I was trying to access the switches in my lab through the terminal server.

You may access the console ports by telneting to the terminal server (username 'admin', su password 'system') and connect from it by "connect local port_x" or by reverse telnet to its IP address, on the port 2000+local_port ( eg. telnet 192.168.0.1 2001 for console port 1). &nbsp;There are a couple of hints you may want to use if you are using the TS as I do: the console ports are set by default act both as DTE or DCE so we need to enforce those as the DTE side.

To avoid these situations you have to set the following options for the ports:

{% highlight bash %}
define port all autostart disabled
define port dtrwait enabled
{% endhighlight %} 

You may always disconnect a hung session on the local console port by:
{% highlight bash %}
logout port x
{% endhighlight %} 

If you are trying to reverse telnet and you get a connection refused it means that the session is probably open or hung.

It's unlikely that you will find this device on the market nowadays but if you happen to find it and need some tweaking advice you can contact me and I'll share my experience.
<img class="size-medium wp-image-116 aligncenter" title="lantronix" src="{{'assets/static/lantronix-300x199.jpg' | prepend: site.baseurl | prepend: site.url }}" alt="" width="300" height="199" />
