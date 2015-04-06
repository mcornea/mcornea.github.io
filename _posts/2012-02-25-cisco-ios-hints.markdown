---
layout: post
title: Cisco IOS disable paging
date: '2012-02-25 22:59:54 +0000'
categories:
- IOS
permalink: ios-disable-paging
---
A nice tip I found about today is how to get rid of the space scrolling when showing a config, log, etc. This may be useful when you want to copy the configuration to a text editor and you don't want to press space so many times to scroll until the end.&nbsp;

{% highlight bash %}
Router#terminal length 0
{% endhighlight %} 

and to have it back to the initial settings:
{% highlight bash %}
Router#terminal length 24
{% endhighlight %} 
