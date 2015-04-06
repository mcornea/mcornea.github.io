---
layout: post
title: Append and prepend using sed
date: '2012-08-25 20:46:33 +0000'
categories:
- Linux
- Automation
permalink: append-and-prepend-using-sed
---
Below is a small example that uses sed to create null routes based on the domain name. It can be useful when the domain name resolves to multiple IP addresses :

{% highlight bash %}
host facebook.com| awk {'print $4'} | sed -n '/^[1-9].*/p' | sed 's/^/ip route /;s/$/ 255.255.255.255 null0/'
The result:
ip route 69.171.234.21 255.255.255.255 null0
ip route 69.171.237.16 255.255.255.255 null0
ip route 69.171.247.21 255.255.255.255 null0
ip route 66.220.149.88 255.255.255.255 null0
ip route 66.220.152.16 255.255.255.255 null0
ip route 66.220.158.70 255.255.255.255 null0
{% endhighlight %} 
