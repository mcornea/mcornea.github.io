---
layout: post
title: Hot Standby Routing Protocol ( HSRP ) Introduction
date: '2012-02-19 21:36:05 +0000'
categories:
- Switching
- Routing
permalink: hot-standby-routing-protocol-hsrp-introduction
---
In the following article I'll make a brief introduction over HSRP and a simple lab topology to show its basic utility. HSRP is Cisco's proprietary protcol which provides non-disruptive IP traffic failover when you've got multiple redundant gateways for you LAN segment. 

___

Let's get back to the basics and remember how 2 hosts communicate on a local subnet: one host broadcasts an ARP request on the local segment, receives the reply, updates its mac address table and then sends frames directly to the learned mac address. When 2 hosts are located on different subnets a router - the default gatway is needed to route packets between the subnets. In this case the hosts operating system checks the destination IP address, sees that it's not part of the local subnet and tries sending the frame to the default gateway. In order to do that the host first sends an ARP request asking for the mac-address associated to IP address of the default gateway and after it receives it caches the mac address and sends all the frames through the default gateway.

Now HSRP comes in place where you have multiple default gateways but still on the hosts you can configure a single default gateway. What HSRP does is that it logically groups the multiple gateways providing a virtual IP and mac address for this group. The group can be formed out of a minimum of 2 devices - one of them will be the active gateway and the other one will be in standby mode . The election is based on a priority value ( 0 - 255 ) and the router with a higher priority is elected as the active one. The default priority is 100 and if both the devices have the same priority the one with a higher IP address becomes the active router.&nbsp;The routers exchange hello messages at regular time intervals so that they can be aware of each others state.

Let's have a look at a simple HSRP topology and see how we can configure basic functionality of the protocol. As you may see we've got 2 LAN segments connected through 2 L3 switches. The HSRP group is configured per interface and as you can see we've got HSRP group 0 for both of our LAN segments. This perfectly valid because the HSRP groups are locally significant only on an interface. The virtual IP address is set per interface and it needs to be the same for all the routers in the group and the virtual mac address is formed by 0000.0c07.acxx where xx is the HSRP group number as 2 hex digits.

In our scenario we will not configure any priority so the elected active router will be the one with the interface which has the higher IP address: GW_2 in our case.

That should be all you have to configure for basic HSRP. More detailed examples to follow where we shall tune the hello timers, configure authentication methods between the group members and try to load balance the traffic with the help of HSRP.

<img class="aligncenter size-large wp-image-123" title="HSRP" src="{{'/public/images/BlankNetworkDiagram-777x1024.png' | prepend: site.baseurl | prepend: site.url }}" alt="" width="550" height="724" />

{% highlight bash %}
GW_2#show standby
FastEthernet0/1 - Group 0
  State is Active
    2 state changes, last state change 00:15:09
  Virtual IP address is 192.168.10.1
  Active virtual MAC address is 0000.0c07.ac00
    Local virtual MAC address is 0000.0c07.ac00 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 1.664 secs
  Preemption disabled
  Active router is local
  Standby router is 192.168.10.2, priority 100 (expires in 10.032 sec)
  Priority 100 (default 100)
  Group name is "hsrp-Fa0/1-0" (default)
FastEthernet0/2 - Group 0
  State is Active
    2 state changes, last state change 00:15:09
  Virtual IP address is 192.168.20.1
  Active virtual MAC address is 0000.0c07.ac00
    Local virtual MAC address is 0000.0c07.ac00 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 2.144 secs
  Preemption disabled
  Active router is local
  Standby router is 192.168.20.2, priority 100 (expires in 9.120 sec)
  Priority 100 (default 100)
  Group name is "hsrp-Fa0/2-0" (default)
GW_2#show run int fa0/1
Building configuration...
Current configuration : 114 bytes
!
interface FastEthernet0/1
 no switchport
 ip address 192.168.10.3 255.255.255.0
 standby 0 ip 192.168.10.1
end
GW_2#show run int fa0/2
Building configuration...
Current configuration : 114 bytes
!
interface FastEthernet0/2
 no switchport
 ip address 192.168.20.3 255.255.255.0
 standby 0 ip 192.168.20.1
end
{% endhighlight %} 

{% highlight bash %}
GW_1#sh standby
FastEthernet0/1 - Group 0
  State is Standby
    4 state changes, last state change 00:15:52
  Virtual IP address is 192.168.10.1
  Active virtual MAC address is 0000.0c07.ac00
    Local virtual MAC address is 0000.0c07.ac00 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.480 secs
  Preemption disabled
  Active router is 192.168.10.3, priority 100 (expires in 10.976 sec)
  Standby router is local
  Priority 100 (default 100)
  Group name is "hsrp-Fa0/1-0" (default)
FastEthernet0/2 - Group 0
  State is Standby
    4 state changes, last state change 00:15:54
  Virtual IP address is 192.168.20.1
  Active virtual MAC address is 0000.0c07.ac00
    Local virtual MAC address is 0000.0c07.ac00 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.224 secs
  Preemption disabled
  Active router is 192.168.20.3, priority 100 (expires in 10.288 sec)
  Standby router is local
  Priority 100 (default 100)
  Group name is "hsrp-Fa0/2-0" (default)
GW_1#sh run int fa0/1
Building configuration...
Current configuration : 114 bytes
!
interface FastEthernet0/1
 no switchport
 ip address 192.168.10.2 255.255.255.0
 standby 0 ip 192.168.10.1
end
GW_1#sh run int fa0/2
Building configuration...
Current configuration : 114 bytes
!
interface FastEthernet0/2
 no switchport
 ip address 192.168.20.2 255.255.255.0
 standby 0 ip 192.168.20.1
end
{% endhighlight %} 
