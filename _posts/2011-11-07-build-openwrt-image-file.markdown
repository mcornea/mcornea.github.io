---
layout: post
title: Build your OpenWRT image file
date: '2011-11-07 04:05:12 +0000'
categories:
- Linux
permalink: build-openwrt-image-file
---
As I said in a previous post I will need an OpenWRT image set with the baud rate of 9600 bps for the console port in order to have my improvised USB-RS232-TTL adapters work. I will show the steps on how to build your image. Please note that I am using Ubuntu as OS but it should work with any other Linux distributions, you just have to install the dependencies.

___

- Get a terminal window and type in:
{% highlight bash %}
$sudo apt-get install build-essential subversion libncurses5-dev zlib1g-dev gawk bison 
gcc flex
$mkdir ~/backfire
$cd ~/backfire
$svn co svn://svn.openwrt.org/openwrt/branches/backfire .
$./scripts/feeds update -a
{% endhighlight %} 

- Prepare build
{% highlight bash %}
$make menuconfig
{% endhighlight %} 

Here you may choose what chipset you are using, the kernel modules and the packages you would like to install. I will choose the ones specific for the router I have : Asus WL500G. Just to be sure I will also install stty - a tool which can be used to set the baud rate of the console port after boot.

{% highlight bash %}
Target System (Broadcom BCM947xx/953xx [2.4])
Target Profile (Generic, Broadcom WiFi (default))
Select all packages by default
Image configuration -->;
      Base system
           busybox (press enter to open hidden menu)
               Configuration
                   Coreutils
                      [*] stty
{% endhighlight %} 

After choosing all the desired settings you have to save your config.

- The next step is to add the 9600 baud rate console option to the kernel:
{% highlight bash %}
$vim ~/backfire/.config
{% endhighlight %} 

and add the following line:
{% highlight bash %}
CONFIG_CMDLINE="root=/dev/mtdblock2 rootfstype=squashfs,jffs2 init=/etc/preinit 
noinitrd console=ttyS0,9600"
{% endhighlight %} 

- Start the build process and wait, it will take a while.
{% highlight bash %}
$make world V=99
{% endhighlight %} 

- When it's finished you'll find the image in ~/backfire/bin and you can upload it through TFTP:
Power your router while holding the reset button
Connect one of the LAN interfaces to your computer and set your computers IP address to 192.168.1.x

{% highlight bash %}
$tftp 192.168.1.1
tftp>binary
tftp>trace 
tftp>put openwrt-brcm-2.4-squashfs.trx
{% endhighlight %} 
<a href="{{'/public/images/Screenshot-1.png' | prepend: site.baseurl | prepend: site.url }}"><img src="{{'/public/images/Screenshot-1.png' | prepend: site.baseurl | prepend: site.url }}" alt="" title="OpenWRT" width="451" height="275" class="aligncenter size-full wp-image-79" /></a>
