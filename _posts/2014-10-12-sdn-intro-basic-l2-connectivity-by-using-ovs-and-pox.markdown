---
layout: post
title: 'SDN Intro: Basic L2 connectivity by using OVS and POX'
date: '2014-10-12 21:35:48 +0000'
categories:
- Linux
- Switching
- Routing
- Virtualization
permalink: sdn-intro-basic-with-ovs-and-pox
---

You've all probably heard about this fancy SDN term that's been passing around in the networking world in the recent years. I'll try to explain below what SDN means for me and what are the benefits of using such a model. 

___

SDN or Software Defined Networking refers to decoupling the data plane from the control plane. The data plane is the process by which the packets are forwarded and it's done by the hardware silicon. The control plane is the process that instructs the data plane how to behave (routing protocols, firewall policies, etc). In a typical networking device we find those 2 planes coupled in a single box. The SDN model says we can separate them and keep the packet forwarding on a box that's got the fast hardware silicon and move the control plane on a general purpose computer. Since the 2 planes are separated they need a means of communication which is OpenFlow. 

OpenFlow is a communications protocol that allows remote manipulation of the devices forwarding tables. Think of this protocol like a standard API that you can consume by running any software. This is great because you can purchase the networking device from a specific vendor and run the controller by your own code, open source project or other proprietary software. In my opinion this provides you the freedom to choose and will push the vendors to create better and better software. Personally I'm a big fan of opensource software and I'm dreaming about the moment when the networking world will be able to get the benefits of a big opensource project. 
Another advantage that SDN brings is a central point that controls the network. Think about how we currently manage the networking devices. Even if the network is a whole that provides services to upper applications, we currently log into each separate device and write some commands that configure services. SDN would save us from doing repetitive and boring tasks such as provisioning vlans.
Enough with the talk, let's start a simple scenario by using the remote-lab.net environment. The lab topology consists of two hosts connected to an OpenvSwitch switch. The OVS switch is connected to an OpenFlow controller running Pox. The controller runs the code that enables L2 connectivity between the hosts but the actual forwarding is done by the OVS switch.
<a href="{{'/public/images/sdn_lab.png' | prepend: site.baseurl | prepend: site.url }}"><img src="{{'/public/images/sdn_lab.png' | prepend: site.baseurl | prepend: site.url }}" alt="sdn_lab" width="686" height="364" class="aligncenter size-full wp-image-284" /></a>

Let's first enable the ports the hosts are connected to, set an openvswitch bridge where the hosts are connected, set the IP addreses on the hosts and check we have connectivity between the hosts. By default the L2 learning mechanism is done by the OpenvSwitch internals. 

{% highlight bash %}
root@ovs01:~>>> ip link set dev eth1 up
root@ovs01:~>>> ip link set dev eth2 up
root@ovs01:~>>> ovs-vsctl add-br sw0
root@ovs01:~>>> ovs-vsctl add-port sw0 eth1
root@ovs01:~>>> ovs-vsctl add-port sw0 eth2
root@host01:~>>> ip l set dev eth1 up
root@host01:~>>> ip addr add 192.168.0.1/24 dev eth1
root@host01:~>>> ip a s dev eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 52:54:00:56:e2:b2 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.1/24 scope global eth1
    inet6 fe80::5054:ff:fe56:e2b2/64 scope link
       valid_lft forever preferred_lft forever
root@host02:~>>> ip l set dev eth1 up
root@host02:~>>> ip addr add 192.168.0.2/24 dev eth1
root@host02:~>>> ip a s dev eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 52:54:00:6e:22:0b brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.2/24 scope global eth1
    inet6 fe80::5054:ff:fe6e:220b/64 scope link
       valid_lft forever preferred_lft forever
root@host02:~>>> ping 192.168.0.1 -c1
PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data.
64 bytes from 192.168.0.1: icmp_req=1 ttl=64 time=0.946 ms
--- 192.168.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.946/0.946/0.946/0.000 ms</code>
{% endhighlight %}

Now let's go to the controller:

{% highlight bash %}
root@ctrl01:~>>> ls
floodlight  pox
{% endhighlight %}

We see here that we have two directories that contain OpenFlow controller code - Floodlight and Pox. We'll choose Pox for our example. Pox is platform for the rapid development and prototyping of network control software using Python. Pox comes with some preinstalled components. One of the components is called forwarding.l2_learning and it does what its name says - make OpenFlow switches act as a type of L2 learning switch. You can find the code in the pox/forwarding/l2_learning.py file. We'll use this component for our example. Let's start it:

{% highlight bash %}
root@ctrl01:~/pox>>> ./pox.py --verbose forwarding.l2_learning
POX 0.2.0 (carp) / Copyright 2011-2013 James McCauley, et al.
DEBUG:core:POX 0.2.0 (carp) going up...
DEBUG:core:Running on CPython (2.7.3/Jan 2 2013 13:56:14)
DEBUG:core:Platform is Linux-3.2.0-4-amd64-x86_64-with-debian-7.4
INFO:core:POX 0.2.0 (carp) is up.
DEBUG:openflow.of_01:Listening on 0.0.0.0:6633
{% endhighlight %}

The next step is to configure OpenvSwitch to use the POX controller. OpenvSwitch has 2 ways of working with an OpenFlow controller - standalone and secure. In standalone mode, if the connection to the controller fails then it will fall back to using its internal logic to install the flows. While in secure mode it will not install any flows if the connection to the controller fails. We'll use the standalone mode with 172.16.18.6 being the IP address of the POX controller.

{% highlight bash %}
root@ovs01:~>>> ovs-vsctl set-fail-mode sw0 standalone
root@ovs01:~>>> ovs-vsctl set-controller sw0 tcp:172.16.18.6:6633
root@ovs01:~>>> ovs-vsctl show
653dfcd6-85a4-4f72-995f-9fa05b5203f9
    Bridge "sw0"
        Controller "tcp:172.16.18.6:6633"
            is_connected: true
        fail_mode: standalone
        Port "sw0"
            Interface "sw0"
                type: internal
        Port "eth2"
            Interface "eth2"
        Port "eth1"
            Interface "eth1"
    ovs_version: "1.9.3"
{% endhighlight %}

We can see the following message on the OpenFlow controller:
{% highlight bash %}
INFO:openflow.of_01:[46-3e-63-ba-e5-46 1] connected
DEBUG:forwarding.l2_learning:Connection [46-3e-63-ba-e5-46 1]
{% endhighlight %}

Let's see what happens when we try to ping one host from the other:

{% highlight bash %}
DEBUG:forwarding.l2_learning:installing flow for 52:54:00:6e:22:0b.2 -> 52:54:00:56:e2:b2.1
DEBUG:forwarding.l2_learning:installing flow for 52:54:00:56:e2:b2.1 -> 52:54:00:6e:22:0b.2
{% endhighlight %}

Check the flows that are installed in the switch. Notice how the flows are defined:
{% highlight bash %}
root@ovs01:~>>> ovs-ofctl dump-flows sw0
NXST_FLOW reply (xid=0x4):
 cookie=0x0, duration=7.348s, table=0, n_packets=7, n_bytes=686, idle_timeout=10, hard_timeout=30, idle_age=1, priority=65535,icmp,in_port=2,vlan_tci=0x0000,dl_src=52:54:00:6e:22:0b,dl_dst=52:54:00:56:e2:b2,nw_src=192.168.0.2,nw_dst=192.168.0.1,nw_tos=0,icmp_type=0,icmp_code=0 actions=output:1
 cookie=0x0, duration=6.329s, table=0, n_packets=6, n_bytes=588, idle_timeout=10, hard_timeout=30, idle_age=1, priority=65535,icmp,in_port=1,vlan_tci=0x0000,dl_src=52:54:00:56:e2:b2,dl_dst=52:54:00:6e:22:0b,nw_src=192.168.0.1,nw_dst=192.168.0.2,nw_tos=0,icmp_type=8,icmp_code=0 actions=output:2
 {% endhighlight %}

You can take any of these headers and manipulate them as you wish. I believe the whole model provides great flexibility and freedom and it will lead to better networking software. 
