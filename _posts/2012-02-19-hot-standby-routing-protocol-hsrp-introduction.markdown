---
layout: post
status: publish
published: true
title: Hot Standby Routing Protocol ( HSRP ) Introduction
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 122
wordpress_url: http://www.remote-lab.net/?p=122
date: '2012-02-19 21:36:05 +0000'
date_gmt: '2012-02-19 18:36:05 +0000'
categories:
- Switching
- Routing
tags: []
comments: []
---
<p>In the following article I'll make a brief introduction over HSRP and a simple lab topology to show its basic utility.</p>
<p>HSRP is Cisco's proprietary protcol which provides non-disruptive IP traffic failover when you've got multiple redundant gateways for you LAN segment. Let's get back to the basics and remember how 2 hosts communicate on a local subnet: one host broadcasts an ARP request on the local segment, receives the reply, updates its mac address table and then sends frames directly to the learned mac address. When 2 hosts are located on different subnets a router - the default gatway is needed to route packets between the subnets. In this case the hosts operating system checks the destination IP address, sees that it's not part of the local subnet and tries sending the frame to the default gateway. In order to do that the host first sends an ARP request asking for the mac-address associated to IP address of the default gateway and after it receives it caches the mac address and sends all the frames through the default gateway.</p>
<p>Now HSRP comes in place where you have multiple default gateways but still on the hosts you can configure a single default gateway. What HSRP does is that it logically groups the multiple gateways providing a virtual IP and mac address for this group. The group can be formed out of a minimum of 2 devices - one of them will be the active gateway and the other one will be in standby mode . The election is based on a priority value ( 0 - 255 ) and the router with a higher priority is elected as the active one. The default priority is 100 and if both the devices have the same priority the one with a higher IP address becomes the active router.&nbsp;The routers exchange hello messages at regular time intervals so that they can be aware of each others state.</p>
<p>Now let's have a look at a simple HSRP topology and see how we can configure basic functionality of the protocol.</p>
<p>As you may see we've got 2 LAN segments connected through 2 L3 switches. The HSRP group is configured per interface and as you can see we've got HSRP group 0 for both of our LAN segments. This perfectly valid because the HSRP groups are locally significant only on an interface. The virtual IP address is set per interface and it needs to be the same for all the routers in the group and the virtual mac address is formed by 0000.0c07.acxx where xx is the HSRP group number as 2 hex digits.<br />
In our scenario we will not configure any priority so the elected active router will be the one with the interface which has the higher IP address: GW_2 in our case.</p>
<p>That should be all you have to configure for basic HSRP. More detailed examples to follow where we shall tune the hello timers, configure authentication methods between the group members and try to load balance the traffic with the help of HSRP.</p>
<p><img class="aligncenter size-large wp-image-123" title="HSRP" src="http://www.remote-lab.net/wp-content/uploads/2012/02/BlankNetworkDiagram-777x1024.png" alt="" width="550" height="724" /></p>
<p><code lang="c[lines-notools]">GW_2#show standby<br />
FastEthernet0/1 - Group 0<br />
  State is Active<br />
    2 state changes, last state change 00:15:09<br />
  Virtual IP address is 192.168.10.1<br />
  Active virtual MAC address is 0000.0c07.ac00<br />
    Local virtual MAC address is 0000.0c07.ac00 (v1 default)<br />
  Hello time 3 sec, hold time 10 sec<br />
    Next hello sent in 1.664 secs<br />
  Preemption disabled<br />
  Active router is local<br />
  Standby router is 192.168.10.2, priority 100 (expires in 10.032 sec)<br />
  Priority 100 (default 100)<br />
  Group name is "hsrp-Fa0/1-0" (default)<br />
FastEthernet0/2 - Group 0<br />
  State is Active<br />
    2 state changes, last state change 00:15:09<br />
  Virtual IP address is 192.168.20.1<br />
  Active virtual MAC address is 0000.0c07.ac00<br />
    Local virtual MAC address is 0000.0c07.ac00 (v1 default)<br />
  Hello time 3 sec, hold time 10 sec<br />
    Next hello sent in 2.144 secs<br />
  Preemption disabled<br />
  Active router is local<br />
  Standby router is 192.168.20.2, priority 100 (expires in 9.120 sec)<br />
  Priority 100 (default 100)<br />
  Group name is "hsrp-Fa0/2-0" (default)<br />
GW_2#show run int fa0/1<br />
Building configuration...</p>
<p>Current configuration : 114 bytes<br />
!<br />
interface FastEthernet0/1<br />
 no switchport<br />
 ip address 192.168.10.3 255.255.255.0<br />
 standby 0 ip 192.168.10.1<br />
end</p>
<p>GW_2#show run int fa0/2<br />
Building configuration...</p>
<p>Current configuration : 114 bytes<br />
!<br />
interface FastEthernet0/2<br />
 no switchport<br />
 ip address 192.168.20.3 255.255.255.0<br />
 standby 0 ip 192.168.20.1<br />
end<br />
</code></p>
<p><code lang="c[lines-notools]">GW_1#sh standby<br />
FastEthernet0/1 - Group 0<br />
  State is Standby<br />
    4 state changes, last state change 00:15:52<br />
  Virtual IP address is 192.168.10.1<br />
  Active virtual MAC address is 0000.0c07.ac00<br />
    Local virtual MAC address is 0000.0c07.ac00 (v1 default)<br />
  Hello time 3 sec, hold time 10 sec<br />
    Next hello sent in 0.480 secs<br />
  Preemption disabled<br />
  Active router is 192.168.10.3, priority 100 (expires in 10.976 sec)<br />
  Standby router is local<br />
  Priority 100 (default 100)<br />
  Group name is "hsrp-Fa0/1-0" (default)<br />
FastEthernet0/2 - Group 0<br />
  State is Standby<br />
    4 state changes, last state change 00:15:54<br />
  Virtual IP address is 192.168.20.1<br />
  Active virtual MAC address is 0000.0c07.ac00<br />
    Local virtual MAC address is 0000.0c07.ac00 (v1 default)<br />
  Hello time 3 sec, hold time 10 sec<br />
    Next hello sent in 0.224 secs<br />
  Preemption disabled<br />
  Active router is 192.168.20.3, priority 100 (expires in 10.288 sec)<br />
  Standby router is local<br />
  Priority 100 (default 100)<br />
  Group name is "hsrp-Fa0/2-0" (default)<br />
GW_1#sh run int fa0/1<br />
Building configuration...</p>
<p>Current configuration : 114 bytes<br />
!<br />
interface FastEthernet0/1<br />
 no switchport<br />
 ip address 192.168.10.2 255.255.255.0<br />
 standby 0 ip 192.168.10.1<br />
end</p>
<p>GW_1#sh run int fa0/2<br />
Building configuration...</p>
<p>Current configuration : 114 bytes<br />
!<br />
interface FastEthernet0/2<br />
 no switchport<br />
 ip address 192.168.20.2 255.255.255.0<br />
 standby 0 ip 192.168.20.1<br />
end<br />
</code></p>
