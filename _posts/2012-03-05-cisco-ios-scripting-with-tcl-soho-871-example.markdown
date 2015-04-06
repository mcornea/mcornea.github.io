---
layout: post
status: publish
published: true
title: Cisco IOS Scripting with TCL - SOHO 871 example
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 111
wordpress_url: http://www.remote-lab.net/?p=111
date: '2012-03-05 02:58:23 +0000'
date_gmt: '2012-03-04 23:58:23 +0000'
categories:
- Uncategorized
tags: []
comments: []
---
<p>Hi guys,</p>
<p>In today's post I will briefly introduce with a simple example how TCL scripting language can be used with the Cisco IOS. Starting with IOS version 12.3(2)T the TCL shell was integrated into Cisco's IOS. Basically it provides a TCL shell environment where you can make use of all the TCL functions, process variables and configure your Cisco device with the help of these. To make an idea of what you can do here's a simple script that would ping all the hosts in a subnet - with the TCL shell it's as simple as defining a counter which will be the host part of the IP address and creating a loop where you are repeating the ping command and incrementing the counter.</p>
<p>Even more with the help of Cisco Embedded Event Manager you can do even more fancy things. The EEM can trigger TCL scripts based on several policies such as SNMP MIB values or MIB variable thresholds or the matching the output of a local syslog message. You can even set your device to send you email messages based on these events which is pretty awesome.</p>
<p>My introductury example consists of a script which will configure a Cisco 871 SOHO router by interactive prompts asking for the hostname, IP addresses, usernames, timezone and setting up basic services like DHCP, NAT and SSH.</p>
<p>The Cisco 871 has 1 x FastEthernet external WAN port a 1 x 4 FastEthernet port switch. In order to run the script you will have to load it into the router's flash memory or run it from a TFTP/HTTP server. This script will automatically set up overload NAT on the external FE interface for all the LAN IP addresses and it also set DHCP providing LAN IP addresses except the ones which are on the excluded list. Below you may find below the output of the script I've written, the resulted running config and also the actual code.</p>
<p>Please let me know what you think about this, in my opinion it's a very powerful tool.</p>
<p>&nbsp;<a href="http://www.remote-lab.net/wp-content/uploads/2012/02/871.txt">871.tcl</a></p>
<p><code lang="c[notools]">Router(tcl)#source flash:871.tcl<br />
Please enter the hostname of this box: 871_SOHO<br />
Please enter the domain name of this box: HOME.NET<br />
Please enter the username you would like to configure for this system: user<br />
Please enter the password for user: pass<br />
Please enter the enable password: cisco<br />
Please enter the WAN IP address: 1.1.1.1<br />
Please enter the WAN netmask: 255.255.255.0<br />
Please enter the default gateway IP address: 1.1.1.2<br />
Please enter the LAN IP address: 192.168.0.1<br />
Please enter the LAN netmask: 255.255.255.240<br />
Configuring the DHCP service...<br />
Please enter the DNS Server IP address: 8.8.8.8<br />
Please enter the excluded IP address from DHCP assignments - separated by spaces 192.168.0.1 192.168.0.2<br />
Configuring NAT ...<br />
Activating SSH ...<br />
Please enter your timezone [ eg. UTC +2 ]: UTC -5<br />
Done...Your box is waiting for some packets!</code></p>
<p><code lang="c[notools]">871_SOHO#sh run<br />
Building configuration...<br />
!<br />
version 12.4<br />
no service pad<br />
service timestamps debug datetime msec<br />
service timestamps log datetime msec<br />
service password-encryption<br />
!<br />
hostname 871_SOHO<br />
!<br />
boot-start-marker<br />
boot-end-marker<br />
!<br />
logging message-counter syslog<br />
logging buffered 25000<br />
enable secret 5 $1$9xIG$BwuXzBtmLsl6acCURFny31<br />
!<br />
aaa new-model<br />
!<br />
!<br />
aaa authentication login default local<br />
!<br />
!<br />
aaa session-id common<br />
clock timezone UTC -5<br />
!<br />
!<br />
dot11 syslog<br />
ip source-route<br />
!<br />
!<br />
ip dhcp excluded-address 192.168.0.1<br />
ip dhcp excluded-address 192.168.0.2<br />
!<br />
ip dhcp pool DHCP<br />
   network 192.168.0.0 255.255.255.240<br />
   default-router 192.168.0.1<br />
   dns-server 8.8.8.8<br />
!<br />
!<br />
ip cef<br />
no ip domain lookup<br />
ip domain name HOME.NET<br />
no ipv6 cef<br />
ntp server 64.6.144.6<br />
!<br />
multilink bundle-name authenticated<br />
!<br />
!<br />
!<br />
username user privilege 15 password 7 15020A1F17<br />
!<br />
!<br />
!<br />
archive<br />
 log config<br />
  hidekeys<br />
!<br />
!<br />
ip ssh version 2<br />
!<br />
!<br />
!<br />
interface FastEthernet0<br />
 switchport access vlan 55<br />
!<br />
interface FastEthernet1<br />
 switchport access vlan 55<br />
!<br />
interface FastEthernet2<br />
 switchport access vlan 55<br />
!<br />
interface FastEthernet3<br />
 switchport access vlan 55<br />
!<br />
interface FastEthernet4<br />
 ip address 1.1.1.1 255.255.255.0<br />
 ip nat outside<br />
 ip virtual-reassembly<br />
 duplex auto<br />
 speed auto<br />
!<br />
interface Vlan1<br />
 no ip address<br />
 shutdown<br />
!<br />
interface Vlan55<br />
 ip address 192.168.0.1 255.255.255.240<br />
 ip nat inside<br />
 ip virtual-reassembly<br />
!<br />
ip forward-protocol nd<br />
ip route 0.0.0.0 0.0.0.0 1.1.1.2<br />
no ip http server<br />
no ip http secure-server<br />
!<br />
!<br />
ip nat inside source list 1 interface FastEthernet4 overload<br />
!<br />
access-list 1 permit 192.168.0.0 0.0.0.15<br />
!<br />
!<br />
!<br />
!<br />
!<br />
control-plane<br />
!<br />
!<br />
line con 0<br />
 logging synchronous<br />
 no modem enable<br />
line aux 0<br />
line vty 0 4<br />
 length 0<br />
 transport input ssh<br />
!<br />
scheduler max-task-time 5000<br />
end       </code></p>
<p><code lang="c[notools]">ios_config "service password-encryption"<br />
ios_config "no service config"<br />
ios_config "line con 0" "logging synchronous"<br />
puts -nonewline "Please enter the hostname of this box: "<br />
flush stdout<br />
set hostname [ gets stdin ]<br />
puts -nonewline "Please enter the domain name of this box: "<br />
flush stdout<br />
set domain [ gets stdin ]<br />
ios_config "ip domain name $domain"<br />
ios_config "hostname $hostname"<br />
ios_config "logging buffered 25000"<br />
ios_config "ip cef"<br />
ios_config "no ip domain lookup"<br />
ios_config "no ip http server"<br />
ios_config "aaa new-model"<br />
ios_config "aaa authentication login default local"<br />
puts -nonewline "Please enter the username you would like to configure for this system: "<br />
flush stdout<br />
set username [ gets stdin ]<br />
puts -nonewline "Please enter the password for $username: "<br />
flush stdout<br />
set password [ gets stdin ]<br />
ios_config "username $username privilege 15 password $password"<br />
puts -nonewline "Please enter the enable password: "<br />
flush stdout<br />
set enable [ gets stdin ]<br />
ios_config "enable secret $enable"<br />
puts -nonewline "Please enter the WAN IP address: "<br />
flush stdout<br />
set wan_ip [ gets stdin ]<br />
puts -nonewline "Please enter the WAN netmask: "<br />
flush stdout<br />
set wan_netmask [ gets stdin ]<br />
ios_config "interface FastEthernet4" "ip address $wan_ip $wan_netmask"<br />
ios_config "interface FastEthernet4" "no shutdown"<br />
puts -nonewline "Please enter the default gateway IP address: "<br />
flush stdout<br />
set def_gw [ gets stdin ]<br />
ios_config "ip route 0.0.0.0 0.0.0.0 $def_gw"<br />
puts -nonewline "Please enter the LAN IP address: "<br />
flush stdout<br />
set lan_ip [ gets stdin ]<br />
puts -nonewline "Please enter the LAN netmask: "<br />
flush stdout<br />
set lan_netmask [ gets stdin ]<br />
ios_config "interface vlan 1" "shutdown"<br />
ios_config "vlan 55" "name Local"<br />
ios_config "interface vlan 55" "ip address $lan_ip $lan_netmask"<br />
ios_config "interface vlan 55" "no shutdown"<br />
ios_config "interface range FastEthernet 0 - 3" "switchport mode access"<br />
ios_config "interface range FastEthernet 0 - 3" "switchport access vlan 55"<br />
puts "Configuring the DHCP service..."<br />
puts -nonewline "Please enter the DNS Server IP address:"<br />
flush stdout<br />
set dns_ip [ gets stdin ]<br />
puts -nonewline "Please enter the excluded IP address from DHCP assignments - separated by spaces "<br />
flush stdout<br />
set excluded [ gets stdin ]<br />
foreach i $excluded {<br />
        ios_config "ip dhcp excluded-address $i"<br />
}<br />
ios_config "ip dhcp pool DHCP" "network $lan_ip $lan_netmask"<br />
ios_config "ip dhcp pool DHCP" "default-route $lan_ip"<br />
ios_config "ip dhcp pool DHCP" "dns-server $dns_ip"<br />
puts "Configuring NAT ..."<br />
set netmask_oct [split $lan_netmask "."]<br />
foreach j $netmask_oct {<br />
        set wildcard_o [expr {255 - $j}]<br />
        lappend wildcard $wildcard_o<br />
        }<br />
set wildcard_o [split $wildcard " "]<br />
set wildcard_o1 [lindex $wildcard_o 0]<br />
set wildcard_o2 [lindex $wildcard_o 1]<br />
set wildcard_o3 [lindex $wildcard_o 2]<br />
set wildcard_o4 [lindex $wildcard_o 3]<br />
set wildcard_mask "$wildcard_o1.$wildcard_o2.$wildcard_o3.$wildcard_o4"<br />
ios_config "access-list 1 permit $lan_ip $wildcard_mask"<br />
ios_config "ip nat inside source list 1 interface FastEthernet4 overload"<br />
ios_config "interface FastEthernet4" "ip nat outside"<br />
ios_config "interface Vlan 55" "ip nat inside"<br />
puts "Activating SSH ..."<br />
ios_config "line vty 0 4" "transport input ssh"<br />
ios_config "crypto key generate rsa general-keys modulus 1024"<br />
ios_config "ip ssh version 2"<br />
ios_config "ntp server 64.6.144.6"<br />
puts -nonewline {Please enter your timezone [ eg. UTC +2 ]:}<br />
flush stdout<br />
set tzone [ gets stdin ]<br />
ios_config "clock timezone $tzone"<br />
puts "Done...Your box is waiting for some packets!"</code></p>
<p><img class="alignright size-thumbnail wp-image-114" title="Code" src="http://www.remote-lab.net/wp-content/uploads/2012/02/Code-150x150.png" alt="" width="19" height="19" /></p>
