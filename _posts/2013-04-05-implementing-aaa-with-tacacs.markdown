---
layout: post
title: Implementing AAA with TACACS+
date: '2013-04-05 23:25:58 +0000'
categories:
- Linux
- IOS
permalink: implementing-aaa-with-tacacs
---
In today's post I will showw how you can do a basic configuration of a TACACS+ Linux server and how to enable the AAA on the networking device.
To start with AAA, stands for Authentication, Authorization and Accounting. The authentication is related to the login process: users and their passwords, authorization describes what each of the users is allowed to do on the device and the accounting part logs what commands the users have issued on the device. All these are implemented as a set of attributes stored in a database that can be located locally on the device or hosted remotely on a TACACS+ or RADIUS server.

___

<a href="{{'assets/static/tacacs.gif' | prepend: site.baseurl | prepend: site.url }}"><img class="aligncenter size-medium wp-image-147" title="tacacs" src="{{'assets/static/tacacs.gif' | prepend: site.baseurl | prepend: site.url }}" alt="" width="300" height="265" /></a>

Installing TACACS+ Server:

I am using Debian Squeeze as OS. TACACS+ comes in Squeeze repositories so installing it is as simple as :
{% highlight bash %}
aptitude install tacacs+
{% endhighlight %} 

Its config file is located at /etc/tacacs+/tac_plus.conf. In order to start, stop, restart it you can use the init scripts.

Let's do a basic configuration and add 2 users on the database: admin with full privileges and user having restricted access:

{% highlight bash %}
#This is the key used by the device to authenticate to the TACACS server.
#We will also set it on the Cisco router
key = testing123 
# This is the admin user having full privileges.
# You may generate the login encrypted password using the tac_pwd tool that comes with the tacacs+ package
user = admin {
default service = permit
login = des RXs9NfUA2sKIE
service = exec {
priv-lvl = 15
}
}
# This is the user having restricted privileges.
# As you may see the default policy is deny for all the commands and we then set to allow the show ip * and show interface * commands, everything else is denied.
user = user {
default service = deny
login = cleartext test
cmd = show
{
permit ip
permit interface
deny .*
}
}
{% endhighlight %} 

Now let’s get to the client side - the Cisco router. We first have to tell the router that it should use TACACS for authentication and the IP address of the server with the authentication key to the server.
{% highlight bash %}
aaa new-model
tacacs-server host $IP_ADDR
tacacs-server key $SERVER_KEY #the same as the one of the server: testing123
{% endhighlight %} 

Next we will configure the authentication methods:

{% highlight bash %}
aaa authentication login vtymethod group tacacs+ enable #uses vtymethod list that first tries the tacacs method and if the server is not reachable users can log in using the enable password
line vty 0 4
   login authentication vtymethod #we tell the router to use the vtymethod list for authenticating the users
{% endhighlight %} 

Now let’s get to the authorization part. We will set it for both privilege level 1 and 15 commands
{% highlight bash %}
aaa authorization config-commands #does authorization for the configuration mode commands
aaa authorization commands 1 default group tacacs+ none #uses the default authorization list which authorizes all the commands through tacacs and if the server is not reachable it won’t do any authorization
aaa authorization commands 15 default group tacacs+ none
{% endhighlight %} 

Accounting:
{% highlight bash %}
aaa accounting exec default start-stop tacacs+
{% endhighlight %} 

In order to have accouting in place you also need to set the accounting log file location on the server:
{% highlight bash %}
accounting file = /var/log/tac_plus.acct
{% endhighlight %} 

Troubleshooting AAA issues can be done using the debug aaa command. Below is an example of the authorization debug for user when trying to enter configuration mode - which is not allowed and when issuing the show ip int brief command allowed:

{% highlight bash %}
Mar 1 03:29:59.859: tty4 AAA/AUTHOR/CMD(4247381793): Port='tty4' list='' service=CMD
*Mar 1 03:29:59.863: AAA/AUTHOR/CMD: tty4(4247381793) user='user'
*Mar 1 03:29:59.863: tty4 AAA/AUTHOR/CMD(4247381793): send AV service=shell
*Mar 1 03:29:59.863: tty4 AAA/AUTHOR/CMD(4247381793): send AV cmd=configure
*Mar 1 03:29:59.863: tty4 AAA/AUTHOR/CMD(4247381793): send AV cmd-arg=terminal
*Mar 1 03:29:59.863: tty4 AAA/AUTHOR/CMD(4247381793): send AV cmd-arg=
*Mar 1 03:29:59.863: tty4 AAA/AUTHOR/CMD(4247381793): found list "default"
*Mar 1 03:29:59.863: tty4 AAA/AUTHOR/CMD(4247381793): Method=tacacs+ (tacacs+)
*Mar 1 03:29:59.863: AAA/AUTHOR/TAC+: (4247381793): user=user
*Mar 1 03:29:59.863: AAA/AUTHOR/TAC+: (4247381793): send AV service=shell
*Mar 1 03:29:59.863: AAA/AUTHOR/TAC+: (4247381793): send AV cmd=configure
*Mar 1 03:29:59.867: AAA/AUTHOR/TAC+: (4247381793): send AV cmd-arg=terminal
*Mar 1 03:29:59.867: AAA/AUTHOR/TAC+: (4247381793): send AV cmd-arg=
*Mar 1 03:30:00.071: TAC+: (-47585503): received author response status = FAIL
*Mar 1 03:30:00.071: AAA/AUTHOR (4247381793): Post authorization status = FAIL
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): Port='tty4' list='' service=CMD
*Mar 1 03:30:23.575: AAA/AUTHOR/CMD: tty4(2242867480) user='user'
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): send AV service=shell
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): send AV cmd=show
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): send AV cmd-arg=ip
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): send AV cmd-arg=interface
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): send AV cmd-arg=brief
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): send AV cmd-arg=
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): found list "default"
*Mar 1 03:30:23.583: tty4 AAA/AUTHOR/CMD(2242867480): Method=tacacs+ (tacacs+)
*Mar 1 03:30:23.583: AAA/AUTHOR/TAC+: (2242867480): user=user
*Mar 1 03:30:23.583: AAA/AUTHOR/TAC+: (2242867480): send AV service=shell
*Mar 1 03:30:23.583: AAA/AUTHOR/TAC+: (2242867480): send AV cmd=show
*Mar 1 03:30:23.583: AAA/AUTHOR/TAC+: (2242867480): send AV cmd-arg=ip
*Mar 1 03:30:23.583: AAA/AUTHOR/TAC+: (2242867480): send AV cmd-arg=interface
*Mar 1 03:30:23.583: AAA/AUTHOR/TAC+: (2242867480): send AV cmd-arg=brief
*Mar 1 03:30:23.583: AAA/AUTHOR/TAC+: (2242867480): send AV cmd-arg=
*Mar 1 03:30:23.795: TAC+: (-2052099816): received author response status = PASS_ADD
*Mar 1 03:30:23.799: AAA/AUTHOR (2242867480): Post authorization status = PASS_ADD
{% endhighlight %} 
