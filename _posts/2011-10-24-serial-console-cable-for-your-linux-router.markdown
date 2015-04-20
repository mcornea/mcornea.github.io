---
layout: post
title: Serial console cable for your Linux router
date: '2011-10-24 22:32:46 +0000'
categories:
- Linux
- Hardware 
permalink: serial-console-cable-for-your-linux-router
---
Today I have been trying to get some working console cables for the Asus WL-500 routers I have. The first thing I tried was using an old Nokia CA-42 data cable which includes an Prolific USB to 3.3V TTL converter but unfortunately it ended in getting a messy input and output. So the other solution I thought of was building my own RS232-to-3.3V TTL converter.

___

Below you may find a scheme of the simplest converter I found. 

<a href="href="{{'/assets/static/ttltors2320kf.jpg' | prepend: site.baseurl | prepend: site.url }}"><img src="{{'/assets/static/ttltors2320kf.jpg' | prepend: site.baseurl | prepend: site.url }}" alt="" title="converter" width="486" height="263" class="aligncenter size-full wp-image-54" /></a>

You will need the following components to build it:

- 2 x BC337 transistors
- 1 x 1.5K resistor
- 1 x 22K resistor
- 1 x 4.7K resistor
- 1 x 3.9K resistor
- 1 x DB9 female connector

This converter did the job for me but only when using it with a computer which had a built-in serial port. When trying it with an USB-to-RS232 converter (Prolific chipset) I got a clean output from the UART port of the router but a messy input. I guess I'll have to try another type of USB-RS232 converter(other chipset than Prolific) or build an USB-to-3.3V TTL converter from scratch.

I am also attaching a document containing a complete list of RS232-to-3.3V TTL converters:

<a href='{{'/assets/static/mt1389-serial-interface-gallery.pdf' | prepend: site.baseurl | prepend: site.url }}'>Converters list</a>

Later Edit: I found a workaround for the messy input when using an USB-to-serial adapter. The default OpenWRT image is compiled with a default baud rate of 115200 bps for the console port. You need to recompile the kernel and use a baud rate of 9600 bps, I will later post a tutorial on how this should be done. 
<a href="{{'/assets/static/IMG_0098.jpg' | prepend: site.baseurl | prepend: site.url }}"><img src="{{'/assets/static/IMG_0098.jpg' | prepend: site.baseurl | prepend: site.url }}" alt="" title="IMG_0098" width="550" height="410" class="aligncenter size-large wp-image-55" /></a>
<a href="{{'/assets/static/IMG_0100.jpg' | prepend: site.baseurl | prepend: site.url }}"><img src="{{'/assets/static/IMG_0100.jpg' | prepend: site.baseurl | prepend: site.url }}" alt="" title="IMG_0100" width="550" height="319" class="aligncenter size-large wp-image-56" /></a> 
