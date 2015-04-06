---
layout: post
status: publish
published: true
title: Running Java Web Start applications on Ubuntu 12.04
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 161
wordpress_url: http://www.remote-lab.net/?p=161
date: '2012-10-06 19:41:33 +0000'
date_gmt: '2012-10-06 16:41:33 +0000'
categories:
- Linux
tags: []
comments: []
---
<p>Hello guys,</p>
<p>In todays post I will present how to install the packages required to run Java web start applications ( .jnlp files) on Ubuntu 12.04.</p>
<p>Most of the KVM(keyboard, video and mouse) devices and several networking management software like Brocade Network Advisor use this kind of apps for remote access.&nbsp;</p>
<p>IcedTea is a web browser plugin running Java applets and Web Start apps. The icedtea-netx package automatically installs all the required Java Openjdk packages.&nbsp;</p>
<p><code lang="c[notools]">aptitude install icedtea-netx icedtea-netx-common icedtea6-plugin</code></p>
