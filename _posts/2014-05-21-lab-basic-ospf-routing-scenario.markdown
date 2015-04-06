---
layout: post
status: publish
published: true
title: Lab - basic OSPF routing scenario
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 258
wordpress_url: https://remote-lab.net/?p=258
date: '2014-05-21 20:25:28 +0000'
date_gmt: '2014-05-21 18:25:28 +0000'
categories:
- Linux
- Switching
- Routing
- IOS
tags: []
comments: []
---
<p>Hello guys,</p>
<p>In the following post we'll see how we can do a basic routing scenario by using the remote-lab.net virtual appliances. Below is the logical diagram of the scenario. Our objective is to esatblish connectivity between the 2 clients: host01 - 10.0.0.10 and host02 - 10.0.1.10. Each of the hosts will be connected to a router that  will be the first hop router for the hosts subnet. The 2 routers will be connected by 2 redundant links. We'll set up OSPF as a routing protocol between the 2 routers that will be used to advertise the clients subnets. </p>
<p>rtr01 is an Arista vEOS and rtr02 is running Vyatta core. host01 and host02 are running Debian.</p>
<p><a href="https://remote-lab.net/wp-content/uploads/2014/05/routing-lab-New-Page.png"><img src="https://remote-lab.net/wp-content/uploads/2014/05/routing-lab-New-Page.png" alt="ospf routing lab logical" width="821" height="729" class="aligncenter size-full wp-image-260" /></a></p>
<p>Now let's get to the physical setup (which is actually virtual as all the components are VMs ). Each system (clients and routers) will be connected to the layer 2 switch (openvswitch running on Linux). We'll need to set up the links that connect the routers to the switch as trunks in order to allow multiple vlans to get through. The ports that connect the clients to the switch will be set as access ports. Please note that in a real world scenario you'll need to connect the routers by 2 different physical links across separate geographical paths to ensure rendundancy. Below is the physical diagram of the setup.</p>
<p><a href="https://remote-lab.net/wp-content/uploads/2014/05/routing-physical-New-Page.png"><img src="https://remote-lab.net/wp-content/uploads/2014/05/routing-physical-New-Page.png" alt="routing lab physical" width="745" height="625" class="aligncenter size-full wp-image-259" /></a></p>
<p>First thing we need to do is to configure the openvswitch. We'll create a bridge that contains all the ports and set up the client ports in access mode. By default all the openvswitch ports are trunks.</p>
<p><code lang="bash[notools]">lab@console01:~>>> ovs01<br />
root@ovs01:~>>> for i in {1..4}; do ip l set dev eth$i up;done<br />
root@ovs01:~>>> ovs-vsctl add-br l2switch<br />
root@ovs01:~>>> ovs-vsctl add-port l2switch eth1 tag=10<br />
root@ovs01:~>>> ovs-vsctl add-port l2switch eth2 tag=20<br />
root@ovs01:~>>> ovs-vsctl add-port l2switch eth3<br />
root@ovs01:~>>> ovs-vsctl add-port l2switch eth4<br />
root@ovs01:~>>> ovs-vsctl show<br />
653dfcd6-85a4-4f72-995f-9fa05b5203f9<br />
    Bridge "l2switch"<br />
        Port "l2switch"<br />
            Interface "l2switch"<br />
                type: internal<br />
        Port "eth3"<br />
            Interface "eth3"<br />
        Port "eth2"<br />
            tag: 20<br />
            Interface "eth2"<br />
        Port "eth4"<br />
            Interface "eth4"<br />
        Port "eth1"<br />
            tag: 10<br />
            Interface "eth1"<br />
    ovs_version: "1.9.3"</code>       </p>
<p>Set up the client interfaces IP addresses and default routes:</p>
<p><code lang="bash[notools]">lab@console01:~>>> host01<br />
root@host01:~>>> ip l set dev eth1 up<br />
root@host01:~>>> ip addr add 10.0.0.10/24 dev eth1<br />
root@host01:~>>> ip route add default via 10.0.0.1</p>
<p>lab@console01:~>>> host02<br />
root@host02:~>>> ip l set dev eth1 up<br />
root@host02:~>>> ip addr add 10.0.1.10/24 dev eth1<br />
root@host01:~>>> ip route add default via 10.0.1.1</code>        </p>
<p>Configure the vlans and SVI IP addresses on the routers:</p>
<p><code lang="c[notools]">lab@console01:~>>> rtr01<br />
rtr01>en<br />
rtr01#configure<br />
rtr01(config)#vlan 10<br />
rtr01(config-vlan-10)#name host01-vlan<br />
rtr01(config-vlan-10)#vlan 100<br />
rtr01(config-vlan-100)#name path1-vlan<br />
rtr01(config-vlan-100)#vlan 200<br />
rtr01(config-vlan-200)#name path2-vlan<br />
rtr01(config)#int ethernet 1<br />
rtr01(config-if-Et1)#switchport mode trunk<br />
rtr01(config-if-Et1)#switchport trunk allowed vlan 10,100,200<br />
rtr01(config-vlan-200)#int vlan 10<br />
rtr01(config-if-Vl10)#ip address 10.0.0.1/24<br />
rtr01(config-if-Et1)#int vlan 100<br />
rtr01(config-if-Vl100)#ip address 192.168.0.1/30<br />
rtr01(config-if-Vl100)#int vlan 200<br />
rtr01(config-if-Vl200)#ip address 192.168.0.5/30                                                         </p>
<p>lab@console01:~>>> rtr02<br />
admin@rtr02:~>>> configure<br />
[edit]<br />
admin@rtr02# set interfaces ethernet eth1 vif 20 address 10.0.1.1/24<br />
[edit]<br />
admin@rtr02# set interfaces ethernet eth1 vif 100 address 192.168.0.2/30<br />
[edit]<br />
admin@rtr02# set interfaces ethernet eth1 vif 200 address 192.168.0.6/30<br />
admin@rtr02# commit<br />
[edit]<br />
admin@rtr02# save<br />
Saving configuration to '/config/config.boot'...<br />
Done</code></p>
<p>Let's do some connectivity tests:</p>
<p><code lang="bash[notools]">rtr01>ping 10.0.0.10<br />
PING 10.0.0.10 (10.0.0.10) 72(100) bytes of data.<br />
80 bytes from 10.0.0.10: icmp_req=1 ttl=64 time=12.9 ms    </p>
<p>rtr01>ping 192.168.0.2<br />
PING 192.168.0.2 (192.168.0.2) 72(100) bytes of data.<br />
80 bytes from 192.168.0.2: icmp_req=1 ttl=64 time=16.7 ms                </p>
<p>rtr01>ping 192.168.0.6<br />
PING 192.168.0.6 (192.168.0.6) 72(100) bytes of data.<br />
80 bytes from 192.168.0.6: icmp_req=1 ttl=64 time=13.2 ms       </p>
<p>admin@rtr02:~>>> ping 10.0.1.10<br />
PING 10.0.1.10 (10.0.1.10) 56(84) bytes of data.<br />
64 bytes from 10.0.1.10: icmp_req=1 ttl=64 time=3.74 ms</code>         </p>
<p>Now that we have the connectivity established on the connected links let's move forward and set up OSPF :</p>
<p><code lang="c[notools]">lab@console01:~>>> rtr01<br />
rtr01>en<br />
rtr01#configure<br />
rtr01(config)#ip routing<br />
rtr01(config)#router ospf 10<br />
rtr01(config-router-ospf)#network 10.0.0.1 0.0.0.0 area 0<br />
rtr01(config-router-ospf)#network 192.168.0.1 0.0.0.0 area 0<br />
rtr01(config-router-ospf)#network 192.168.0.5 0.0.0.0 area 0 </p>
<p>lab@console01:~>>> rtr02<br />
admin@rtr02:~>>> configure<br />
[edit]<br />
admin@rtr02#<br />
admin@rtr02# set protocols ospf area 0 network 10.0.1.0/24<br />
[edit]<br />
admin@rtr02# set protocols ospf area 0 network 192.168.0.0/30<br />
[edit]<br />
admin@rtr02# set protocols ospf area 0 network 192.168.0.4/30<br />
admin@rtr02# commit</code></p>
<p>Once that we have the ospf configuration done we can proceed and check the OSPF neighbor status on the 2 routers. We'll see that we have 2 equal cost paths to the hosts subnets. This means that the routers will load balance the packets through the 2 links.      </p>
<p><code lang="c[notools]">rtr01#show ip ospf neighbor<br />
Neighbor ID     VRF    Pri   State            Dead Time   Address         Interface<br />
172.16.18.5     default    1   FULL/BDR         00:00:31    192.168.0.6     Vlan200<br />
172.16.18.5     default    1   FULL/DR          00:00:31    192.168.0.2     Vlan100    </p>
<p>rtr01#show ip route 10.0.1.0/24<br />
Codes: C - connected, S - static, K - kernel,<br />
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,<br />
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,<br />
       N2 - OSPF NSSA external type2, B I - iBGP, B E - eBGP,<br />
       R - RIP, I - ISIS, A B - BGP Aggregate, A O - OSPF Summary                                                                                                                             </p>
<p> O      10.0.1.0/24 [110/20] via 192.168.0.2, Vlan100<br />
                             via 192.168.0.6, Vlan200                     </p>
<p>admin@rtr02:~>>> show ip ospf neighbor </p>
<p>    Neighbor ID Pri State           Dead Time Address         Interface            RXmtL RqstL DBsmL<br />
192.168.0.5       1 Full/Backup       36.750s 192.168.0.1     eth1.100:192.168.0.2     0     0     0<br />
192.168.0.5       1 Full/DR           36.960s 192.168.0.5     eth1.200:192.168.0.6     0     0     0 </p>
<p>admin@rtr02:~>>> show ip route 10.0.0.0/24<br />
Routing entry for 10.0.0.0/24<br />
  Known via "ospf", distance 110, metric 20, best<br />
  Last update 00:04:42 ago<br />
  * 192.168.0.1, via eth1.100<br />
  * 192.168.0.5, via eth1.200</code>                 </p>
<p>What if we want to set one of the links as primary ? We need to set a higher cost for the secondary link. At this time both of the paths have a cost of 20 (10 for the network segment that connects the 2 routers + 10 for the host network segment). Let'schoose the vlan 200 as secondary and increase the cost for the secondary link to 11.</p>
<p><code lang="c[notools]">rtr01#configure<br />
rtr01(config)#int vlan 200<br />
rtr01(config-if-Vl200)#ip ospf cost 11<br />
rtr01(config-if-Vl200)#show ip route 10.0.1.0/24<br />
Codes: C - connected, S - static, K - kernel,<br />
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,<br />
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,<br />
       N2 - OSPF NSSA external type2, B I - iBGP, B E - eBGP,<br />
       R - RIP, I - ISIS, A B - BGP Aggregate, A O - OSPF Summary                                                                                                                             </p>
<p> O      10.0.1.0/24 [110/20] via 192.168.0.2, Vlan100</code>      </p>
<p>We can see that the routing table only contains the vlan 100 path. What happens if vlan 100 goes down ? </p>
<p><code lang="c[notools]">rtr01(config)#int vlan 100<br />
rtr01(config-if-Vl100)#shut<br />
rtr01(config-if-Vl100)#show ip route 10.0.1.0/24<br />
Codes: C - connected, S - static, K - kernel,<br />
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,<br />
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,<br />
       N2 - OSPF NSSA external type2, B I - iBGP, B E - eBGP,<br />
       R - RIP, I - ISIS, A B - BGP Aggregate, A O - OSPF Summary                                                                                                                             </p>
<p> O      10.0.1.0/24 [110/21] via 192.168.0.6, Vlan200</code>          </p>
<p>We can see that the vlan 200 path is installed in the routing table with a cost of 21 ( 11 + 10).</p>
<p>I hope this post is useful for getting an idea of how you can use the virtual lab. Please let me know if you any questions.                                                             </p>
