---
layout: post
status: publish
published: true
title: Blacklisting Linux kernel modules using the grub command line
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 155
wordpress_url: http://www.remote-lab.net/?p=155
date: '2012-09-06 00:10:02 +0000'
date_gmt: '2012-09-05 21:10:02 +0000'
categories:
- Linux
tags: []
comments: []
---
<p>
In order the prevent certain modules from being loaded by the kernel using the grub command line you need to pass them as a parameter to the &nbsp;kernel line using the below syntax. It can be useful when you cannot access the OS file system so that you can edit a blacklist with the modules you don't want to be loaded&nbsp;</p>
<p><code lang="c[notools]"> module_name.blacklist=yes</code></p>
