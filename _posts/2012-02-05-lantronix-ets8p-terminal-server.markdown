---
layout: post
status: publish
published: true
title: " Lantronix ETS8P Terminal Server"
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 115
wordpress_url: http://www.remote-lab.net/?p=115
date: '2012-02-05 20:14:04 +0000'
date_gmt: '2012-02-05 17:14:04 +0000'
categories:
- Uncategorized
tags: []
comments:
- id: 85
  author: Robert Rittenhouse
  author_email: robert@tadstech.com
  author_url: ''
  date: '2012-02-17 20:33:06 +0000'
  date_gmt: '2012-02-17 17:33:06 +0000'
  content: "I am configuring one of these for a Cisco lab. My issue is: When I telnet
    into the device's physical port 2 (IP:3002) I am getting the router prompt twice
    when I hit enter as well as everything I type. It looks like it's echo'ng and
    my client is too but I can't seem to fix it. Do you know how to fix/turn off the
    echo on the Lantronix side?\r\n\r\nIf I telnet into port 7000 on the Lantronix
    device and then use connect local..etc. it works fine. It seems to just be a setting
    on it or something.\r\n\r\nThanks."
- id: 86
  author: marius
  author_email: marius@remote-lab.net
  author_url: ''
  date: '2012-02-18 02:48:11 +0000'
  date_gmt: '2012-02-17 23:48:11 +0000'
  content: |-
    Hi Robert,

    Try to reverse telnet on the device's IP address on port 2000 + local port ( e.g. telnet 192.168.0.1 2002 for physical port 2 ).
    Let me know if that worked for you.

    Marius
- id: 6791
  author: Alan
  author_email: alan@ajservices.biz
  author_url: ''
  date: '2013-05-24 09:50:07 +0000'
  date_gmt: '2013-05-24 07:50:07 +0000'
  content: Hi, got one of these and could do with some help configuring. basically
    i use it for a cisco lab like yourself but keep getting no sessions active errors.
    Alan
- id: 7190
  author: John Joy
  author_email: ccieguy@gmail.com
  author_url: ''
  date: '2013-06-21 14:56:36 +0000'
  date_gmt: '2013-06-21 12:56:36 +0000'
  content: Has anybody tried to connect Juniper SRXs with Lantronix ETS? I am getting
    "connection refused" message when tried to connect the SRX with "connect local
    port_x".
- id: 7200
  author: John Joy
  author_email: ccieguy@gmail.com
  author_url: ''
  date: '2013-06-22 20:18:32 +0000'
  date_gmt: '2013-06-22 18:18:32 +0000'
  content: "I bought my Lantronix ETS16PR for $42.50 from EBay. However I have spent
    well over 4 weeks to make it worked so you don't have to. Over all, this 20+ years
    old  TS does work. Now I can finally go back to the important stuff, my study.
    \r\n\r\nFor reference, please see below:\r\n\r\n\\ Basic Setting \\\r\n\r\nset
    privileged\r\n\r\ndefine port all autostart disabled ---&gt; thanks, Marius\r\ndefine
    port dtrwait enabled          ---&gt; thanks, Marius\r\nset server ipaddress 192.168.1.168\r\ndef
    server ipaddress 192.168.1.168\r\nset server subnet mask 255.255.255.0\r\ndef
    server subnet mask 255.255.255.0\r\ndef server gateway 192.168.1.1\r\ndefine port
    all autostart disabled\r\ndefine port dtrwait enabled \r\ndef port 2-16 autoprompt
    dis\r\nset port 2-16 autoprompt dis\r\ndef port 2-16 broadcast dis\r\nset port
    2-16 broadcast dis\r\ndef port 2-16 loss not dis\r\nset port 2-16 loss not dis\r\ndef
    port 2-16 ver dis\r\nset port 2-16 ver dis\r\ndef port 2-16 remote conf dis\r\nset
    port 2-16 remote conf dis\r\ndef port 2-16 telnet pad dis\r\nset port 2-16 telnet
    pad dis\r\ndef port 2-16 flow cts\r\nset port 2-16 flow cts\r\ndef port 2-16 termtype
    \"vt100\"\r\nset port 2-16 termtype \"vt100\"\r\ndefine port 2-16 acccess remote\r\ndefine
    port 2-16 break disabled\r\ndefine ports 2-16 access remote\r\ndefine ports 2-16
    break disabled\r\nDEFINE PORT 2-16 LOCAL SWITCH ^X\r\nDEFINE PORT 2-16 FORWARD
    SWITCH ^F\r\nDEFINE PORT 2-16 BACKWARD SWITCH ^B\r\nDEFINE PORT 2-16 BREAK LOCAL\r\nlogout
    port 2-16\r\nlogout\r\n\r\n\\ Menu Setting \\\r\n\r\nset privileged\r\n\r\ndefine
    menu 1 \"Juniper SRX-1\" \"connect local port_11\"\r\ndefine menu 2 \"Juniper
    SRX-2\" \"connect local port_12\"\r\ndefine menu 3 \"Juniper SRX-3\" \"connect
    local port_13\"\r\ndefine menu 4 \"Juniper SRX-4\" \"connect local port_14\"\r\ndefine
    menu 5 \"Juniper SRX-5\" \"connect local port_15\"\r\ndefine menu 6 \"Juniper
    EX2-1\" \"connect local port_16\"\r\ndefine menu 7 \"Cisco Catalyst 3560\" \"connect
    local port_10\"\r\ndefine menu 8 \"ETS Prompt\" \"exit\"\r\ndefine menu 9 \"Logout\"
    \"logout\"\r\ndefine port 10-16 menu enable\r\ndefine port 0 menu enable --&gt;
    without it, the menu is useless!\r\nlogout port 10-16\r\nlogout\r\n\r\n\\ Validation
    \\\r\n\r\nshow server\r\nshow menu\r\nlist menu\r\n\r\n\\ Last Note \\\r\n\r\nIf
    the hung session persists, the culprit, most likely, is the rollover cable."
- id: 7201
  author: John Joy
  author_email: ccieguy@gmail.com
  author_url: ''
  date: '2013-06-22 20:21:06 +0000'
  date_gmt: '2013-06-22 18:21:06 +0000'
  content: DEFINE PORT 2-16 BREAK LOCAL --&gt; please change "local" to "remote" if
    you want to access them remotely.
---
<p>Hi guys,</p>
<p>I recently got a terminal server for my lab because as the number of devices grows it was difficult to manage them all by USB console adapters. So i found a cheap and quite old terminal server - Lantronix ETS8P - a terminal server with 8 serial RJ45 ports and 1 x 10Mbps ethernet interface. It's actually that old that it doesn't even support SSH, you can remote access it just by telnet. Anyways, I was very excited when I got it and I got right away a console cable on my laptop and connected to it - &nbsp;I was thinking I will have it configured in a couple of minutes but in the end it took me a couple of more minutes to have it ready.&nbsp;</p>
<p>I configured an IP address for it, prepared 8 rollover cables for connecting to Cisco devices, connected my devices and after that there I was trying to access the switches in my lab through the terminal server.</p>
<p>You may access the console ports by telneting to the terminal server (username 'admin', su password 'system') and connect from it by "connect local port_x" or by reverse telnet to its IP address, on the port 2000+local_port ( eg. telnet 192.168.0.1 2001 for console port 1). &nbsp;There are a couple of hints you may want to use if you are using the TS as I do: the console ports are set by default act both as DTE or DCE so we need to enforce those as the DTE side.</p>
<p>To avoid these situations you have to set the following options for the ports:</p>
<pre>define port all autostart disabled
define port dtrwait enabled&nbsp;</pre>
<p>You may always disconnect a hung session on the local console port by:</p>
<pre>logout port x</pre>
<p>If you are trying to reverse telnet and you get a connection refused it means that the session is probably open or hung.</p>
<p>It's unlikely that you will find this device on the market nowadays but if you happen to find it and need some tweaking advice you can contact me and I'll share my experience.</p>
<p style="text-align: center;"><img class="size-medium wp-image-116 aligncenter" title="lantronix" src="http://www.remote-lab.net/wp-content/uploads/2012/02/lantronix-300x199.jpg" alt="" width="300" height="199" />&nbsp;</p>
