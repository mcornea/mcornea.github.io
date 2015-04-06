---
layout: post
status: publish
published: true
title: Implementing AAA with TACACS+
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 146
wordpress_url: http://www.remote-lab.net/?p=146
date: '2013-04-05 23:25:58 +0000'
date_gmt: '2013-04-05 21:25:58 +0000'
categories:
- Linux
- IOS
tags: []
comments: []
---
<p>Hello guys,</p>
<p>I am sorry for not posting lately but I have been very busy with a new job and lots of new stuff that I needed to learn. As you can see remote-lab.net has a new look and I am also working on a new lab structure and booking system.</p>
<p>In today's post I will present how you can do a basic configuration of a TACACS+ Linux server and how to enable the AAA on the networking device.</p>
<p>To start with AAA stands for Authentication, Authorization and Accounting. The authentication is related to the login process: users and their passwords, authorization describes what each of the users is allowed to do on the device and the accounting part logs what commands the users have issued on the device. All these are implemented as a set of attributes stored in a database that can be located locally on the device or hosted remotely on a TACACS+ or RADIUS server.</p>
<p><a href="http://www.remote-lab.net/wp-content/uploads/2012/08/tacacs.gif"><img class="aligncenter size-medium wp-image-147" title="tacacs" src="http://www.remote-lab.net/wp-content/uploads/2012/08/tacacs-300x265.gif" alt="" width="300" height="265" /></a></p>
<p>Installing TACACS+ Server:</p>
<p>I am using Debian Squeeze as OS. TACACS+ comes in Squeeze repositories so installing it is as simple as :</p>
<p><code lang="c[notools]">aptitude install tacacs+</code></p>
<p>Its config file is located at /etc/tacacs+/tac_plus.conf. In order to start, stop, restart it you can use the init scripts.</p>
<p>Let's do a basic configuration and add 2 users on the database: admin with full privileges and user having restricted access:</p>
<p><code lang="c[notools]">#This is the key used by the device to authenticate to the TACACS server.<br />
#We will also set it on the Cisco router<br />
key = testing123 </p>
<p># This is the admin user having full privileges.<br />
# You may generate the login encrypted password using the tac_pwd tool that comes with the tacacs+ package</p>
<p>user = admin {<br />
default service = permit<br />
login = des RXs9NfUA2sKIE<br />
service = exec {<br />
priv-lvl = 15<br />
}<br />
}</p>
<p># This is the user having restricted privileges.<br />
# As you may see the default policy is deny for all the commands and we then set to allow the show ip * and show interface * commands, everything else is denied.</p>
<p>user = user {<br />
default service = deny<br />
login = cleartext test<br />
cmd = show<br />
{<br />
permit ip<br />
permit interface<br />
deny .*<br />
}<br />
}</p>
<p></code></p>
<p>Now let’s get to the client side - the Cisco router. We first have to tell the router that it should use TACACS for authentication and the IP address of the server with the authentication key to the server.</p>
<p><code lang="c[notools]">aaa new-model<br />
tacacs-server host $IP_ADDR<br />
tacacs-server key $SERVER_KEY #the same as the one of the server: testing123</code></p>
<p>Next we will configure the authentication methods:</p>
<p><code lang="c[notools]">aaa authentication login vtymethod group tacacs+ enable #uses vtymethod list that first tries the tacacs method and if the server is not reachable users can log in using the enable password<br />
line vty 0 4<br />
   login authentication vtymethod #we tell the router to use the vtymethod list for authenticating the users</code></p>
<p>Now let’s get to the authorization part. We will set it for both privilege level 1 and 15 commands</p>
<p><code lang="c[notools]">aaa authorization config-commands #does authorization for the configuration mode commands<br />
aaa authorization commands 1 default group tacacs+ none #uses the default authorization list which authorizes all the commands through tacacs and if the server is not reachable it won’t do any authorization<br />
aaa authorization commands 15 default group tacacs+ none</code></p>
<p>Accounting:</p>
<p><code lang="c[notools]">aaa accounting exec default start-stop tacacs+</p>
<p>#In order to have accouting in place you also need to set the accounting log file location on the server:</p>
<p>accounting file = /var/log/tac_plus.acct</code></p>
<p>Troubleshooting AAA issues can be done using the debug aaa command. Below is an example of the authorization debug for user when trying to enter configuration mode - which is not allowed and when issuing the show ip int brief command allowed:</p>
<p><code lang="c[notools]">Mar 1 03:29:59.859: tty4 AAA/AUTHOR/CMD(4247381793): Port='tty4' list='' service=CMD<br />
*Mar 1 03:29:59.863: AAA/AUTHOR/CMD: tty4(4247381793) user='user'<br />
*Mar 1 03:29:59.863: tty4 AAA/AUTHOR/CMD(4247381793): send AV service=shell<br />
*Mar 1 03:29:59.863: tty4 AAA/AUTHOR/CMD(4247381793): send AV cmd=configure<br />
*Mar 1 03:29:59.863: tty4 AAA/AUTHOR/CMD(4247381793): send AV cmd-arg=terminal<br />
*Mar 1 03:29:59.863: tty4 AAA/AUTHOR/CMD(4247381793): send AV cmd-arg=<br />
*Mar 1 03:29:59.863: tty4 AAA/AUTHOR/CMD(4247381793): found list "default"<br />
*Mar 1 03:29:59.863: tty4 AAA/AUTHOR/CMD(4247381793): Method=tacacs+ (tacacs+)<br />
*Mar 1 03:29:59.863: AAA/AUTHOR/TAC+: (4247381793): user=user<br />
*Mar 1 03:29:59.863: AAA/AUTHOR/TAC+: (4247381793): send AV service=shell<br />
*Mar 1 03:29:59.863: AAA/AUTHOR/TAC+: (4247381793): send AV cmd=configure<br />
*Mar 1 03:29:59.867: AAA/AUTHOR/TAC+: (4247381793): send AV cmd-arg=terminal<br />
*Mar 1 03:29:59.867: AAA/AUTHOR/TAC+: (4247381793): send AV cmd-arg=<br />
*Mar 1 03:30:00.071: TAC+: (-47585503): received author response status = FAIL<br />
*Mar 1 03:30:00.071: AAA/AUTHOR (4247381793): Post authorization status = FAIL<br />
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): Port='tty4' list='' service=CMD<br />
*Mar 1 03:30:23.575: AAA/AUTHOR/CMD: tty4(2242867480) user='user'<br />
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): send AV service=shell<br />
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): send AV cmd=show<br />
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): send AV cmd-arg=ip<br />
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): send AV cmd-arg=interface<br />
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): send AV cmd-arg=brief<br />
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): send AV cmd-arg=<br />
*Mar 1 03:30:23.575: tty4 AAA/AUTHOR/CMD(2242867480): found list "default"<br />
*Mar 1 03:30:23.583: tty4 AAA/AUTHOR/CMD(2242867480): Method=tacacs+ (tacacs+)<br />
*Mar 1 03:30:23.583: AAA/AUTHOR/TAC+: (2242867480): user=user<br />
*Mar 1 03:30:23.583: AAA/AUTHOR/TAC+: (2242867480): send AV service=shell<br />
*Mar 1 03:30:23.583: AAA/AUTHOR/TAC+: (2242867480): send AV cmd=show<br />
*Mar 1 03:30:23.583: AAA/AUTHOR/TAC+: (2242867480): send AV cmd-arg=ip<br />
*Mar 1 03:30:23.583: AAA/AUTHOR/TAC+: (2242867480): send AV cmd-arg=interface<br />
*Mar 1 03:30:23.583: AAA/AUTHOR/TAC+: (2242867480): send AV cmd-arg=brief<br />
*Mar 1 03:30:23.583: AAA/AUTHOR/TAC+: (2242867480): send AV cmd-arg=<br />
*Mar 1 03:30:23.795: TAC+: (-2052099816): received author response status = PASS_ADD<br />
*Mar 1 03:30:23.799: AAA/AUTHOR (2242867480): Post authorization status = PASS_ADD</code></p>
