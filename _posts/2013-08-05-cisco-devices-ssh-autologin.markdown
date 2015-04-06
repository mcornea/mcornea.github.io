---
layout: post
status: publish
published: true
title: Cisco devices SSH autologin
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 202
wordpress_url: https://remote-lab.net/?p=202
date: '2013-08-05 23:20:26 +0000'
date_gmt: '2013-08-05 21:20:26 +0000'
categories:
- Linux
- IOS
tags: []
comments: []
---
<p>Below is a quick expect script that enables you to autologin on Cisco devices with a predefined clear text password by using an OpenSSH client :</p>
<p><code lang="c[notools]">#!/usr/bin/expect<br />
set password "yourverycleartextpassword"<br />
spawn ssh [lindex $argv 0] -o PreferredAuthentications=keyboard-interactive,password<br />
expect "Password: "<br />
send "$password\r"<br />
interact<br />
exit</code></p>
