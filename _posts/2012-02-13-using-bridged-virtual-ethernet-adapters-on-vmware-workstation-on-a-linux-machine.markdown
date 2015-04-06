---
layout: post
status: publish
published: true
title: Using VMware Workstation bridged virtual ethernet adapters on a Linux machine
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 118
wordpress_url: http://www.remote-lab.net/?p=118
date: '2012-02-13 23:09:51 +0000'
date_gmt: '2012-02-13 20:09:51 +0000'
categories:
- Linux
- Virtualization
tags: []
comments: []
---
<p>I've recently installed VMware Workstation on my Ubuntu machine and today I was triyng to run &nbsp;an ESXi guest VM on it. I set the network interface for the ESXi VM to be bridged with one of the physical NICs which was connected to my LAN. The installation went smooth but when trying to ping the ESXi VM no reply was coming back. After investigating it looked like it was an issue with the permissions for the virtual&nbsp;Ethernet&nbsp;adapters.</p>
<p>When you edit the virtual adapters using the Virtual Network Editor you need root permissions as in any Linux environment only the root can edit the network interfaces. But when you run the VMware Workstation application, you run it as your user without root privileges. The VMware Workstation process tries to set the virtual network adapter to promiscous mode and in order to do that it requires privileged rights.</p>
<p>In order to set the virtual adapters into&nbsp;promiscuous&nbsp;mode without running the VMware process as root we'll have to modify the permissions for each of the virtual adapter we'd like to use. Run the following command to set the permissions to read and write for the owner, group and others for /dev/vmnet0:</p>
<p><code lang="bash[notools]">sudo chmod a+rw /dev/vmnet0</code></p>
<p>If you have more than one adapter as I have on my machine you can use a simple script to save some time - sets the permissions for /dev/vmnet0 /dev/vmnet1 ... /dev/vmnet9</p>
<p><code lang="bash[notools]">#!/bin/bash</p>
<p>for (( i=0; i<=9; i++ ))<br />
do<br />
 chmod a+rw /dev/vmnet$i<br />
done</code>&nbsp;</p>
<p><img class="aligncenter size-full wp-image-119" style="border-style: initial; border-color: initial;" title="bridge" src="http://www.remote-lab.net/wp-content/uploads/2012/02/bridge.png" alt="" width="224" height="190" /></p>
