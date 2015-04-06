---
layout: post
status: publish
published: true
title: Linux Servers Cluster using HeartBeat
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 106
wordpress_url: http://www.remote-lab.net/?p=106
date: '2012-01-15 22:26:52 +0000'
date_gmt: '2012-01-15 19:26:52 +0000'
categories:
- Linux
tags: []
comments: []
---
<p>Hello guys,</p>
<p>Today I've been working to set up a High Availability cluster made up of 2 virtual instances of CentOS Servers using Heartbeat. The cluster will be using one of the servers as primary and the second one as backup. The basic idea behing Heartbeat is that the backup server will be continuously pinging the primary server (using broadcast, multicast or unicast packets). In the eventuality that the active server goes down the backup server sets up a floating IP address, sends a gratious ARP message informing the others hosts on the local network segment that the mac-address corresponding to the floating IP has changed. The floating IP address is an IP address shared by the active and standby servers but it's assigned just to one of them at a time.</p>
<p>Heartbeat setup is pretty straight forward, I'll describe in the following lines how you can install it on a CentOS system.</p>
<p>1. Download the latest epel-release rpm from</p>
<pre>[root@server1 ~]# wget http://download.fedora.redhat.com/pub/epel/6/i386/epel-release-6-5.noarch.rpm
[root@server1 ~]# rpm -uvh epel-release-6-5.noarch.rpm</pre>
<p>2. Install Heartbeat packages:</p>
<pre>[root@server1 ~]# yum install heartbeat</pre>
<p>3. The Heartbeat config files are authkeys, ha.cf and haresources. We have to move thos into /etc/ha.d/ directory and edit them according to our setup.</p>
<pre>[root@server1 ~]# cp /usr/share/doc/heartbeat-3.0.4/authkeys /etc/ha.d/
[root@server1 ~]# cp /usr/share/doc/heartbeat-3.0.4/ha.cf /etc/ha.d/
[root@server1 ~]# cp /usr/share/doc/heartbeat-3.0.4/haresources /etc/ha.d/</pre>
<p>3.1 Edit authkeys with sha1 as authentication method:</p>
<pre>[root@server1 ~]# vim /etc/ha.d/authkeys</pre>
<pre>auth 2
2 sha1 pass</pre>
<p>Change the permission for /etc/ha.d/authkeys to "600":</p>
<pre> [root@server1 ~]# chmod 600 /etc/ha.d/authkeys</pre>
<p>3.2. Now let's edit the ha.cf configuration file:</p>
<pre>[root@server1 ~]# vim /etc/ha.d/ha.cf</pre>
<pre>logfile /var/log/ha-log #File to write log messages to
logfacility local0 #Facility to use for syslog()/logger
keepalive 1 #how long between heartbeats?
deadtime 5 #how long-to-declare-host-dead?
bcast eth0 #What interfaces to broadcast heartbeats over?
udpport 694 #What UDP port to use for bcast/ucast communication?
auto_failback on #if set to on the resource will automatically fail back 
                  to the primary server once it restores
node server1 #the hostname of the primary server - must match uname -n
node server2 #the hostname of the secondary server</pre>
<p>3.3 Edit the haresources file - it must contain the hostname of the primary server and the floating virtual IP shared by the 2 servers:</p>
<pre>[root@server1 ~]# vim /etc/ha.d/haresources</pre>
<pre>server1 IPaddr::10.0.0.20/255.255.255.0</pre>
<p>Now let's edit the Linux hosts file so that we map each of the hostnames to their corresponind IP address:</p>
<pre>[root@server1 ~]# vim /etc/hosts
 10.0.0.11 server1
 10.0.0.12 server2</pre>
<p>The steps above must be repeated on Server2, please note that the configuration on both the machines in the cluster must match.<br />
After setting up the config on the machines we are ready to start the Heartbeat service.</p>
<p>4. Starting the Heartbeat service:</p>
<pre>[root@server1 ~]# /etc/init.d/heartbeat start
Starting High-Availability services: IPaddr[7248]: INFO: Resource is stopped
Done.</pre>
<p>Now let's test configuration, here comes the tricky part where I encountered some issues with my setup:</p>
<p><a href="http://www.remote-lab.net/wp-content/uploads/2012/02/cluster_setup.png"><img class="aligncenter size-medium wp-image-107" title="cluster_setup" src="http://www.remote-lab.net/wp-content/uploads/2012/02/cluster_setup-300x295.png" alt="" width="300" height="295" /></a></p>
<p>So we've got server1 (10.0.0.11) and server2 (10.0.0.12) running on 2 virtual machine instances. I'm starting some ping packets from my PC (10.0.0.5) to the floating IP address - 10.0.0.20. I'm checking the ARP table of the PC and I'm seeing that 10.0.0.20 is associated to the mac address of server1, the primary node. Next, I shutdown interface eth0 of server1 and as configured in the ha.cf file I'm expecting for the ping packets to fail just for a couple of seconds more than 5 seconds.<br />
Unfortunately I don't get this result with my setup, the ICMP replies start to appear after aproximately 40 seconds.</p>
<p>From what I'm seeing the gratious ARP message which had to be sent by Heartbeat to inform that the HW MAC address corresponding to 10.0.0.20 has changed doesn't reach my PC. So the PC waits until the OS flushes the ARP entry for 10.0.0.20 and then broadcasts another ARP request for 10.0.0.20.</p>
<p>One thing I don't understand is that before sending a broadcast ARP in the network to discover who has 10.0.0.20 my PC sends 3 unicast ARP messages to the MAC Address of the host which went down asking for 10.0.0.20.</p>
<p><a href="http://www.remote-lab.net/wp-content/uploads/2012/02/Screenshot.png"><img class="aligncenter size-large wp-image-108" title="ARP" src="http://www.remote-lab.net/wp-content/uploads/2012/02/Screenshot-1024x110.png" alt="" width="550" height="59" /></a></p>
<p>If you have got any ideas please let me know how I can improve this behavior.</p>
