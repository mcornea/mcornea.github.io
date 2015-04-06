---
layout: post
title: Blacklisting Linux kernel modules using the grub command line
date: '2012-09-06 00:10:02 +0000'
categories:
- Linux
permalink: blacklisting-kernel-modules
---
In order the prevent certain modules from being loaded by the kernel using the grub command line you need to pass them as a parameter to the kernel line using the below syntax. It can be useful when you cannot access the OS file system so that you can edit a blacklist with the modules you don't want to be loaded

{% highlight bash %}
$module_name.blacklist=yes
{% endhighlight %} 

