---
layout: post
status: publish
published: true
title: Build your OpenWRT image file
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 78
wordpress_url: http://www.remote-lab.net/?p=78
date: '2011-11-07 04:05:12 +0000'
date_gmt: '2011-11-07 01:05:12 +0000'
categories:
- Linux
tags: []
comments: []
---
<p>Hey guys,</p>
<p>As I said in a previous post I will need an OpenWRT image set with the baud rate of 9600 bps for the console port in order to have my improvised USB-RS232-TTL adapters work. I will present below the steps on how to build your image. Please note that I am using Ubuntu as OS but it should work with any other Linux distributions, you just have to install the dependencies.</p>
<p>1. Get a terminal window and type in:</p>
<pre style="padding-bottom: 0.75em; background-color: #eeeeee; padding-left: 1.5em; padding-right: 1.5em; font-size: 12px; padding-top: 0.75em; border: #dddddd 1px solid;">$sudo apt-get install build-essential subversion libncurses5-dev zlib1g-dev gawk bison 
gcc flex
$mkdir ~/backfire
$cd ~/backfire
$svn co svn://svn.openwrt.org/openwrt/branches/backfire .
$./scripts/feeds update -a</pre>
<p>2.</p>
<pre style="padding-bottom: 0.75em; background-color: #eeeeee; padding-left: 1.5em; padding-right: 1.5em; font-size: 12px; padding-top: 0.75em; border: #dddddd 1px solid;">$make menuconfig</pre>
<p>Here you may choose what chipset you are using, the kernel modules and the packages you would like to install. I will choose the ones specific for the router I have : Asus WL500G. Just to be sure I will also install stty - a tool which can be used to set the baud rate of the console port after boot.</p>
<pre style="padding-bottom: 0.75em; background-color: #eeeeee; padding-left: 1.5em; padding-right: 1.5em; font-size: 12px; padding-top: 0.75em; border: #dddddd 1px solid;">Target System (Broadcom BCM947xx/953xx [2.4])
Target Profile (Generic, Broadcom WiFi (default))
Select all packages by default
Image configuration --&gt;;
      Base system
           busybox (press enter to open hidden menu)
               Configuration
                   Coreutils
                      [*] stty</pre>
<p>After choosing all the desired settings you have to save your config.</p>
<p>3. The next step is to add the 9600 baud rate console option to the kernel:</p>
<pre style="padding-bottom: 0.75em; background-color: #eeeeee; padding-left: 1.5em; padding-right: 1.5em; font-size: 12px; padding-top: 0.75em; border: #dddddd 1px solid;">$vim ~/backfire/.config</pre>
<p>and add the following line:</p>
<pre style="padding-bottom: 0.75em; background-color: #eeeeee; padding-left: 1.5em; padding-right: 1.5em; font-size: 12px; padding-top: 0.75em; border: #dddddd 1px solid;">
CONFIG_CMDLINE="root=/dev/mtdblock2 rootfstype=squashfs,jffs2 init=/etc/preinit 
noinitrd console=ttyS0,9600"
</pre>
<p>4. Start the build process and wait, it will take a while.</p>
<pre style="padding-bottom: 0.75em; background-color: #eeeeee; padding-left: 1.5em; padding-right: 1.5em; font-size: 12px; padding-top: 0.75em; border: #dddddd 1px solid;">$make world V=99</pre>
<p>5. When it's finished you'll find the image in ~/backfire/bin and you can upload it through TFTP:</p>
<pre style="padding-bottom: 0.75em; background-color: #eeeeee; padding-left: 1.5em; padding-right: 1.5em; font-size: 12px; padding-top: 0.75em; border: #dddddd 1px solid;">Power your router while holding the reset button
Connect one of the LAN interfaces to your computer and set your computers IP address 
to 192.168.1.x
$tftp 192.168.1.1
tftp&gt;binary
tftp&gt;trace 
tftp&gt;put openwrt-brcm-2.4-squashfs.trx</pre>
<p><a href="http://www.remote-lab.net/wp-content/uploads/2011/11/Screenshot-1.png"><img src="http://www.remote-lab.net/wp-content/uploads/2011/11/Screenshot-1.png" alt="" title="OpenWRT" width="451" height="275" class="aligncenter size-full wp-image-79" /></a></p>
