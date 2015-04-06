---
layout: post
title: Cisco devices SSH autologin
date: '2013-08-05 23:20:26 +0000'
categories:
- Linux
- IOS
permalink: cisco-devices-ssh-autologin
---
Below is a quick expect script that enables you to autologin on Cisco devices with a predefined clear text password by using an OpenSSH client :

___

{% highlight bash %}
#!/usr/bin/expect
set password "yourverycleartextpassword"
spawn ssh [lindex $argv 0] -o PreferredAuthentications=keyboard-interactive,password
expect "Password: "
send "$password\r"
interact
exit
{% endhighlight %} 

