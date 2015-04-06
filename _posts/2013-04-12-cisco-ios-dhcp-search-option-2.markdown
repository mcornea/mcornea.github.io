---
layout: post
title: Cisco IOS DHCP search option
date: '2013-04-12 23:50:25 +0000'
categories:
- Linux
- IOS
- Python
permalink: cisco-ios-dhcp-search-option
---
I was looking today for a way to set my home Cisco router to push multiple domains in the DHCP search list. I found this very useful post written by Jonathan Perkin: <a href="http://www.perkin.org.uk/posts/serving-multiple-dns-search-domains-in-ios-dhcp.html">http://www.perkin.org.uk/posts/serving-multiple-dns-search-domains-in-ios-dhcp.html</a> where he explains how we can achieve this by using Cisco’s hex sequence in the search option. He also provides a nice python script that converts the domain ASCII string to hex sequence.

___

Thank you very much, Jonathan, very useful info.

{% highlight python %}
cat ios-search.py
#!/usr/bin/python
import sys
hexlist = []
for domain in sys.argv[1:]:
    for part in domain.split("."):
        hexlist.append("%02x" % len(part))
        for c in part:
            hexlist.append(c.encode("hex"))
    hexlist.append("00")
print "".join([(".%s" % (x) if i and not i % 2 else x) \
    for i, x in enumerate(hexlist)])
{% endhighlight %} 

{% highlight bash %}
marius@remoteur:~>>> ./ios-search.py domain.net domain.org domain.com
0664.6f6d.6169.6e03.6e65.7400.0664.6f6d.6169.6e03.6f72.6700.0664.6f6d.6169.6e03.636f.6d00
{% endhighlight %} 

{% highlight bash %}
c881.remote-lab.net#show run | s ip dhcp pool
ip dhcp pool HomeLeases
   network 10.0.0.0 255.255.255.0
   default-router 10.0.0.1
   domain-name remote-lab.net
   dns-server 8.8.8.8 8.8.4.4
   option 119 hex 0664.6f6d.6169.6e03.6e65.7400.0664.6f6d.6169.6e03.6f72.6700.0664.6f6d.6169.6e03.636f.6d00
{% endhighlight %} 

{% highlight bash %}
marius@remoteur:~>>> grep search /etc/resolv.conf
search remote-lab.net domain.net domain.org domain.com
{% endhighlight %} 
