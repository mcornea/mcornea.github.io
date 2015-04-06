---
layout: post
status: publish
published: true
title: Deprecated Linux networking commands
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 148
wordpress_url: http://www.remote-lab.net/?p=148
date: '2012-08-07 00:05:39 +0000'
date_gmt: '2012-08-06 21:05:39 +0000'
categories:
- Linux
tags: []
comments:
- id: 165
  author: Brendan
  author_email: bchoi9999@gmail.com
  author_url: http://labswitch.blogspot.com
  date: '2012-08-07 05:27:16 +0000'
  date_gmt: '2012-08-07 02:27:16 +0000'
  content: '"ip link show" is a very good command :)'
---
<p>
I am trying to get rid of the bad habit of using deprecated Linux networking tools like ifconfig, route or netstat.</p>
<p>I will start this list with currently maintained packages and common commands I am using and I'll keep it updated.</p>
<p>You can post in the comment area which are your favorite commands so that we can make this list complete.</p>
<p><code lang="c[notools]">ip a #Display information related to all network interfaces<br />
ip a show dev $interface #Display information related to a specific interface<br />
ip addr add $ip_address/$netmask dev $interface #Add IP address<br />
ip addr del $ip_address/$netmask dev $interface #Delete IP address<br />
ip r #Display routing table<br />
ip r add $network_address/$netmask via $next_hop #Add a static route<br />
ip r del $network_address/$netmask #Delete a static route:<br />
ip n #Display ARP table:<br />
ip l set dev $interface {up|down} #Self explanatory<br />
ss -a #Display listening and non-listening sockets<br />
ss -l #Display listening sockets<br />
lsof -i #Display all opened network sockets and their corresponding process<br />
ip link show dev $interface #Display interface link layer info</code></p>
