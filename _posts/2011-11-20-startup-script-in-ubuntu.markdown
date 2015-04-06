---
layout: post
title: Startup scripts in Ubuntu and Debian based Linux distributions
date: '2011-11-20 00:43:37 +0000'
date_gmt: '2011-11-19 21:43:37 +0000'
categories:
- Linux
- Debian
permalink: startup-script-in-ubuntu
---
Here are 2 simple steps which help you to run a script when your Ubuntu machine boots:

1. Edit your script and add it in /etc/init.d/
2. Run update-rc.d script which creates a link to your script on each of the runlevels  with a set priority.

{% highlight bash %}
root@virtual:/etc/init.d# update-rc.d vbox.sh defaults 85
update-rc.d: warning: /etc/init.d/vbox.sh missing LSB information
update-rc.d: see &lt;http://wiki.debian.org/LSBInitScripts
 Adding system startup for /etc/init.d/vbox.sh ...
   /etc/rc0.d/K85vbox.sh -&gt; ../init.d/vbox.sh
   /etc/rc1.d/K85vbox.sh -&gt; ../init.d/vbox.sh
   /etc/rc6.d/K85vbox.sh -&gt; ../init.d/vbox.sh
   /etc/rc2.d/S85vbox.sh -&gt; ../init.d/vbox.sh
   /etc/rc3.d/S85vbox.sh -&gt; ../init.d/vbox.sh
   /etc/rc4.d/S85vbox.sh -&gt; ../init.d/vbox.sh
   /etc/rc5.d/S85vbox.sh -&gt; ../init.d/vbox.sh
{% endhighlight %} 

