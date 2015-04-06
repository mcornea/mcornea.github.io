---
layout: post
title: Lab - basic OSPF routing scenario
date: '2014-05-21 20:25:28 +0000'
categories:
- Linux
- Switching
- Routing
- vEOS
permalink: basic-ospf-routing-lab
---

In this post post we'll see how we can do a basic routing scenario by using the remote-lab.net virtual appliances. Below is the logical diagram of the scenario. Our objective is to esatblish connectivity between the 2 clients: host01 - 10.0.0.10 and host02 - 10.0.1.10. 

___

Each of the hosts will be connected to a router that  will be the first hop router for the hosts subnet. The 2 routers will be connected by 2 redundant links. We'll set up OSPF as a routing protocol between the 2 routers that will be used to advertise the clients subnets. 
rtr01 is an Arista vEOS and rtr02 is running Vyatta core. host01 and host02 are running Debian.
<a href="{{'assets/static/routing-lab-New-Page.png' | prepend: site.baseurl | prepend: site.url }}"><img src="{{'assets/static/routing-lab-New-Page.png' | prepend: site.baseurl | prepend: site.url }}" alt="ospf routing lab logical" width="821" height="729" class="aligncenter size-full wp-image-260" /></a>

Now let's get to the physical setup (which is actually virtual as all the components are VMs ). Each system (clients and routers) will be connected to the layer 2 switch (openvswitch running on Linux). We'll need to set up the links that connect the routers to the switch as trunks in order to allow multiple vlans to get through. The ports that connect the clients to the switch will be set as access ports. Please note that in a real world scenario you'll need to connect the routers by 2 different physical links across separate geographical paths to ensure rendundancy. Below is the physical diagram of the setup.

<a href="{{'assets/static/routing-physical-New-Page.png' | prepend: site.baseurl | prepend: site.url }}"><img src="{{'assets/static/routing-physical-New-Page.png' | prepend: site.baseurl | prepend: site.url }}" alt="routing lab physical" width="745" height="625" class="aligncenter size-full wp-image-259" /></a>

First thing we need to do is to configure the openvswitch. We'll create a bridge that contains all the ports and set up the client ports in access mode. By default all the openvswitch ports are trunks.

{% highlight bash %}
lab@console01:~>>> ovs01
root@ovs01:~>>> for i in {1..4}; do ip l set dev eth$i up;done
root@ovs01:~>>> ovs-vsctl add-br l2switch
root@ovs01:~>>> ovs-vsctl add-port l2switch eth1 tag=10
root@ovs01:~>>> ovs-vsctl add-port l2switch eth2 tag=20
root@ovs01:~>>> ovs-vsctl add-port l2switch eth3
root@ovs01:~>>> ovs-vsctl add-port l2switch eth4
root@ovs01:~>>> ovs-vsctl show
653dfcd6-85a4-4f72-995f-9fa05b5203f9
    Bridge "l2switch"
        Port "l2switch"
            Interface "l2switch"
                type: internal
        Port "eth3"
            Interface "eth3"
        Port "eth2"
            tag: 20
            Interface "eth2"
        Port "eth4"
            Interface "eth4"
        Port "eth1"
            tag: 10
            Interface "eth1"
    ovs_version: "1.9.3"
{% endhighlight %} 

Set up the client interfaces IP addresses and default routes:

{% highlight bash %}
lab@console01:~>>> host01
root@host01:~>>> ip l set dev eth1 up
root@host01:~>>> ip addr add 10.0.0.10/24 dev eth1
root@host01:~>>> ip route add default via 10.0.0.1
lab@console01:~>>> host02
root@host02:~>>> ip l set dev eth1 up
root@host02:~>>> ip addr add 10.0.1.10/24 dev eth1
root@host01:~>>> ip route add default via 10.0.1.1
{% endhighlight %} 

Configure the vlans and SVI IP addresses on the routers:

{% highlight bash %}
lab@console01:~>>> rtr01
rtr01>en
rtr01#configure
rtr01(config)#vlan 10
rtr01(config-vlan-10)#name host01-vlan
rtr01(config-vlan-10)#vlan 100
rtr01(config-vlan-100)#name path1-vlan
rtr01(config-vlan-100)#vlan 200
rtr01(config-vlan-200)#name path2-vlan
rtr01(config)#int ethernet 1
rtr01(config-if-Et1)#switchport mode trunk
rtr01(config-if-Et1)#switchport trunk allowed vlan 10,100,200
rtr01(config-vlan-200)#int vlan 10
rtr01(config-if-Vl10)#ip address 10.0.0.1/24
rtr01(config-if-Et1)#int vlan 100
rtr01(config-if-Vl100)#ip address 192.168.0.1/30
rtr01(config-if-Vl100)#int vlan 200
rtr01(config-if-Vl200)#ip address 192.168.0.5/30                                                         
lab@console01:~>>> rtr02
admin@rtr02:~>>> configure
[edit]
admin@rtr02# set interfaces ethernet eth1 vif 20 address 10.0.1.1/24
[edit]
admin@rtr02# set interfaces ethernet eth1 vif 100 address 192.168.0.2/30
[edit]
admin@rtr02# set interfaces ethernet eth1 vif 200 address 192.168.0.6/30
admin@rtr02# commit
[edit]
admin@rtr02# save
Saving configuration to '/config/config.boot'...
Done
{% endhighlight %} 

Let's do some connectivity tests:

{% highlight bash %}
rtr01>ping 10.0.0.10
PING 10.0.0.10 (10.0.0.10) 72(100) bytes of data.
80 bytes from 10.0.0.10: icmp_req=1 ttl=64 time=12.9 ms    
rtr01>ping 192.168.0.2
PING 192.168.0.2 (192.168.0.2) 72(100) bytes of data.
80 bytes from 192.168.0.2: icmp_req=1 ttl=64 time=16.7 ms                
rtr01>ping 192.168.0.6
PING 192.168.0.6 (192.168.0.6) 72(100) bytes of data.
80 bytes from 192.168.0.6: icmp_req=1 ttl=64 time=13.2 ms       
admin@rtr02:~>>> ping 10.0.1.10
PING 10.0.1.10 (10.0.1.10) 56(84) bytes of data.
64 bytes from 10.0.1.10: icmp_req=1 ttl=64 time=3.74 ms
{% endhighlight %} 

Now that we have the connectivity established on the connected links let's move forward and set up OSPF :

{% highlight bash %}
lab@console01:~>>> rtr01
rtr01>en
rtr01#configure
rtr01(config)#ip routing
rtr01(config)#router ospf 10
rtr01(config-router-ospf)#network 10.0.0.1 0.0.0.0 area 0
rtr01(config-router-ospf)#network 192.168.0.1 0.0.0.0 area 0
rtr01(config-router-ospf)#network 192.168.0.5 0.0.0.0 area 0 
lab@console01:~>>> rtr02
admin@rtr02:~>>> configure
[edit]
admin@rtr02#
admin@rtr02# set protocols ospf area 0 network 10.0.1.0/24
[edit]
admin@rtr02# set protocols ospf area 0 network 192.168.0.0/30
[edit]
admin@rtr02# set protocols ospf area 0 network 192.168.0.4/30
admin@rtr02# commit
{% endhighlight %} 

Once that we have the ospf configuration done we can proceed and check the OSPF neighbor status on the 2 routers. We'll see that we have 2 equal cost paths to the hosts subnets. This means that the routers will load balance the packets through the 2 links.      

{% highlight bash %}
rtr01#show ip ospf neighbor
Neighbor ID     VRF    Pri   State            Dead Time   Address         Interface
172.16.18.5     default    1   FULL/BDR         00:00:31    192.168.0.6     Vlan200
172.16.18.5     default    1   FULL/DR          00:00:31    192.168.0.2     Vlan100    
rtr01#show ip route 10.0.1.0/24
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B I - iBGP, B E - eBGP,
       R - RIP, I - ISIS, A B - BGP Aggregate, A O - OSPF Summary                                                                                                                             
 O      10.0.1.0/24 [110/20] via 192.168.0.2, Vlan100
                             via 192.168.0.6, Vlan200                     
admin@rtr02:~>>> show ip ospf neighbor 
    Neighbor ID Pri State           Dead Time Address         Interface            RXmtL RqstL DBsmL
192.168.0.5       1 Full/Backup       36.750s 192.168.0.1     eth1.100:192.168.0.2     0     0     0
192.168.0.5       1 Full/DR           36.960s 192.168.0.5     eth1.200:192.168.0.6     0     0     0 
admin@rtr02:~>>> show ip route 10.0.0.0/24
Routing entry for 10.0.0.0/24
  Known via "ospf", distance 110, metric 20, best
  Last update 00:04:42 ago
  * 192.168.0.1, via eth1.100
  * 192.168.0.5, via eth1.200
{% endhighlight %} 

What if we want to set one of the links as primary ? We need to set a higher cost for the secondary link. At this time both of the paths have a cost of 20 (10 for the network segment that connects the 2 routers + 10 for the host network segment). Let'schoose the vlan 200 as secondary and increase the cost for the secondary link to 11.

{% highlight bash %}
rtr01#configure
rtr01(config)#int vlan 200
rtr01(config-if-Vl200)#ip ospf cost 11
rtr01(config-if-Vl200)#show ip route 10.0.1.0/24
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B I - iBGP, B E - eBGP,
       R - RIP, I - ISIS, A B - BGP Aggregate, A O - OSPF Summary                                                                                                                             
 O      10.0.1.0/24 [110/20] via 192.168.0.2, Vlan100
{% endhighlight %} 

We can see that the routing table only contains the vlan 100 path. What happens if vlan 100 goes down ? 

{% highlight bash %}
rtr01(config)#int vlan 100
rtr01(config-if-Vl100)#shut
rtr01(config-if-Vl100)#show ip route 10.0.1.0/24
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B I - iBGP, B E - eBGP,
       R - RIP, I - ISIS, A B - BGP Aggregate, A O - OSPF Summary                                                                                                                             
 O      10.0.1.0/24 [110/21] via 192.168.0.6, Vlan200
{% endhighlight %} 

We can see that the vlan 200 path is installed in the routing table with a cost of 21 ( 11 + 10).

I hope this post is useful for getting an idea of how you can use the virtual lab. Please let me know if you any questions.                                                             
