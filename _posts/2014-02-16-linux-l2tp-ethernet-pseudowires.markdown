---
layout: post
title: Linux L2TP ethernet pseudowires
date: '2014-02-16 23:29:21 +0000'
categories:
- Linux
- Switching
- Routing
- Virtualization
permalink: linux-l2tp-ethernet-pseudowires
---
This post describes how you can create L2TP ethernet pseudowires by using Linux kernel's L2TP drivers and the "ip" utility of iproute2.  L2TP is a protocol that tunnels one or more sessions over an IP tunnel. It is commonly used for VPNs (L2TP/IPSec) and by ISPs to tunnel subscriber PPP sessions over an IP network infrastructure. With L2TPv3, it is also useful as a Layer-2 tunneling infrastructure. 

___

Our topology consists of 5 virtual machines running Debian Wheezy with Linux Kernel 3.2.0. Our scenario objective is to ensure L2 connectivity between HOST01 and HOST02. In order to accomplish this we'll create a tunnel between TUNNEL01 and TUNNEL02 that will encapsulate packets coming from the 192.168.0.0/24 network. The INTERNET box just acts as a router for the packets exchanged by TUNNEL01 and TUNNEL02. 
<a href="{{ 'assets/static/L2TP.png' | prepend: site.baseurl | prepend: site.url }}"><img src="{{ 'assets/static/L2TP.png' | prepend: site.baseurl | prepend: site.url }}" alt="L2TP" width="851" height="408" class="aligncenter size-full wp-image-238" /></a>

First thing let's make sure IP forwarding is enabled on all the machines that'll forward packets : TUNNEL0[1-2], INTERNET:

{% highlight bash %}
root@tun01:~# echo 1 > /proc/sys/net/ipv4/ip_forward
root@tun02:~# echo 1 > /proc/sys/net/ipv4/ip_forward
root@tun-inet:~# echo 1 > /proc/sys/net/ipv4/ip_forward
{% endhighlight %} 

Next thing to do is to establish L3 connectivity between TUNNEL01 and TUNNEL02 by setting static routes:

{% highlight bash %}
root@tun01:~# ip route add 2.2.2.0/30 via 1.1.1.1
root@tun02:~# ip route add 1.1.1.0/30 via 2.2.2.1
root@tun01:~# ping -c1 2.2.2.2
PING 2.2.2.2 (2.2.2.2) 56(84) bytes of data.
64 bytes from 2.2.2.2: icmp_req=1 ttl=63 time=1.03 ms
root@tun02:~# ping -c1 1.1.1.2
PING 1.1.1.2 (1.1.1.2) 56(84) bytes of data.
64 bytes from 1.1.1.2: icmp_req=1 ttl=63 time=1.20 ms
{% endhighlight %} 

Once we've established L3 connectivity between the tunnel endpoints we can proceed to creating the tunnels. Before configuring the tunnels we need to load the L2TPv3 ethernet pseudowire kernel module:

{% highlight bash %}
root@tun01:~# modprobe l2tp_eth
root@tun01:~# ip l2tp add tunnel tunnel_id 1000 peer_tunnel_id 2000 encap udp local 1.1.1.2 remote 2.2.2.2 udp_sport 6000 udp_dport 5000
root@tun01:~# ip l2tp add session tunnel_id 1000 session_id 3000 peer_session_id 4000
root@tun01:~# ip l2tp show tunnel
Tunnel 1000, encap UDP
  From 1.1.1.2 to 2.2.2.2
  Peer tunnel 2000
  UDP source / dest ports: 6000/5000
root@tun01:~# ip l2tp show session
Session 3000 in tunnel 1000
  Peer session 4000, tunnel 2000
  interface name: l2tpeth0
  offset 0, peer offset 0
root@tun02:~# modprobe l2tp_eth
root@tun02:~# ip l2tp add tunnel tunnel_id 2000 peer_tunnel_id 1000 encap udp local 2.2.2.2 remote 1.1.1.2 udp_sport 5000 udp_dport 6000
root@tun02:~# ip l2tp add session tunnel_id 2000 session_id 4000 peer_session_id 3000
root@tun02:~# ip l2tp show tunnel
Tunnel 2000, encap UDP
  From 2.2.2.2 to 1.1.1.2
  Peer tunnel 1000
  UDP source / dest ports: 5000/6000
root@tun02:~# ip l2tp show session
Session 4000 in tunnel 2000
  Peer session 3000, tunnel 1000
  interface name: l2tpeth0
  offset 0, peer offset 0
{% endhighlight %} 

We'll notice a new interface has been created with a MTU of 1488 (1500bytes Ethernet MTU - 12bytes L2TP header): 

{% highlight bash %}
root@tun01:~# ip a s dev l2tpeth0
5: l2tpeth0: <BROADCAST,MULTICAST> mtu 1488 qdisc noop state DOWN qlen 1000
    link/ether 1a:8f:6e:04:3f:a3 brd ff:ff:ff:ff:ff:ff
root@tun02:~# ip a s dev l2tpeth0
5: l2tpeth0: <BROADCAST,MULTICAST> mtu 1488 qdisc noop state DOWN qlen 1000
    link/ether 3e:d8:00:8c:d0:a2 brd ff:ff:ff:ff:ff:ff
{% endhighlight %} 

Next thing to do is install bridge-utils and bridge the l2tp interface with the network interface that's attached to the LAN segment we want to extend over the tunnel, in our case it's called eth1:

{% highlight bash %}
root@tun01:~# aptitude install bridge-utils
root@tun02:~# aptitude install bridge-utils
root@tun01:~# brctl addbr l2tp
root@tun02:~# brctl addbr l2tp
root@tun01:~# brctl addif l2tp eth1 l2tpeth0
root@tun02:~# brctl addif l2tp eth1 l2tpeth0
root@tun01:~# brctl show
bridge name	        bridge id	    STP enabled	     interfaces
l2tp		  8000.1a8f6e043fa3           no		            eth1 l2tpeth0
root@tun01:~# ip l set dev l2tpeth0 up
root@tun01:~# ip l set dev l2tp up
root@tun02:~# ip l set dev l2tpeth0 up
root@tun02:~# ip l set dev l2tp up
{% endhighlight %} 

Now we should be up and running and have L2 connectivity over the L2TP pseudowire:

{% highlight bash %}
root@host01:~# ping -c5 192.168.0.4
PING 192.168.0.4 (192.168.0.4) 56(84) bytes of data.
64 bytes from 192.168.0.4: icmp_req=1 ttl=64 time=3.85 ms
64 bytes from 192.168.0.4: icmp_req=2 ttl=64 time=1.93 ms
64 bytes from 192.168.0.4: icmp_req=3 ttl=64 time=1.91 ms
64 bytes from 192.168.0.4: icmp_req=4 ttl=64 time=1.87 ms
64 bytes from 192.168.0.4: icmp_req=5 ttl=64 time=1.89 ms
{% endhighlight %} 

Now let's go further to the transport layer and do a TCP throughput test:
{% highlight bash %}
root@host02:~# nuttcp -S
root@host01:~# nuttcp 192.168.0.4
nuttcp-t: v6.1.2: Error: server not ACKing data
{% endhighlight %} 

Looks that we've got a problem. It appears that the TCP session was established with a MSS of 1460bytes. This would cause packets fragmentation through the tunnel interface as the packets would exceed the maximum MTU on the Ethernet link of 1500bytes. Even though the connection should establish correctly and the transfer should run just fine fragmentation could case performance issues due to encapsulation overhead. 

<a href="https://remote-lab.net/wp-content/uploads/2014/02/mss.png"><img src="https://remote-lab.net/wp-content/uploads/2014/02/mss-1024x438.png" alt="mss" width="960" height="410" class="aligncenter size-large wp-image-239" /></a>

I thought of lowering the MTU of the internal interfaces and enforcing the MSS for the TCP connections so that all the packets that are transmitted across the tunnel interfaces do not exceed 1500bytes.
Now let's do some basic maths and find out what's the maximum TCP payload, also known as the maximum segment size in the TCP header:

{% highlight bash %}
1500B=20B(IP_HEADER)+8B(UDP_HEADER)+12B(L2TP_HEADER)+14B(ETH_HEADER)+20B(IP_HEADER)+20B(TCP_HEADER)+PAYLOAD => PAYLOAD=1406B

root@tun01:~# ip link set eth1 mtu 1446
root@tun01:~# iptables -A FORWARD -p tcp --tcp-flags SYN,RST SYN -m tcpmss --mss 1406:1536 -j TCPMSS --set-mss 1406
root@tun02:~# ip link set eth1 mtu 1446
root@tun02:~# iptables -A FORWARD -p tcp --tcp-flags SYN,RST SYN -m tcpmss --mss 1406:1536 -j TCPMSS --set-mss 1406
root@host01:~# nuttcp 192.168.0.4
   45.7992 MB /  10.05 sec =   38.2292 Mbps 15 %TX 30 %RX 0 retrans 1.89 msRTT
{% endhighlight %} 

All the virtual machines are running on my laptop so I guess it's a decent throughput as the CPU seems to be the bottleneck. 

I hope you find this post useful. I am going to continue it in a future post by configuring L2TP over IPsec. 

