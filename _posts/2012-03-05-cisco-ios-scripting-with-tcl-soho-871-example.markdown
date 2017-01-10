---
layout: post
title: Cisco IOS Scripting with TCL - Cisco 871 example
date: '2012-03-05 02:58:23 +0000'
categories:
- IOS
- Automation
permalink: ios-scripting-with-tcl-cisco-871
---
In today's post I will briefly introduce with a simple example how TCL scripting language can be used with the Cisco IOS. Starting with IOS version 12.3(2)T the TCL shell was integrated into Cisco's IOS. Basically it provides a TCL shell environment where you can make use of all the TCL functions, process variables and configure your Cisco device with the help of these. 

___

To make an idea of what you can do here's a simple script that would ping all the hosts in a subnet - with the TCL shell it's as simple as defining a counter which will be the host part of the IP address and creating a loop where you are repeating the ping command and incrementing the counter. Even more with the help of Cisco Embedded Event Manager you can do even more fancy things. The EEM can trigger TCL scripts based on several policies such as SNMP MIB values or MIB variable thresholds or the matching the output of a local syslog message. You can even set your device to send you email messages based on these events which is pretty awesome.

My example consists of a script which will configure a Cisco 871 router by interactive prompts asking for the hostname, IP addresses, usernames, timezone and setting up basic services like DHCP, NAT and SSH.
The Cisco 871 has 1 x FastEthernet external WAN port a 1 x 4 FastEthernet port switch. In order to run the script you will have to load it into the router's flash memory or run it from a TFTP/HTTP server. This script will automatically set up overload NAT on the external FE interface for all the LAN IP addresses and it also set DHCP providing LAN IP addresses except the ones which are on the excluded list. Below you may find below the output of the script I've written, the resulted running config and also the actual code.

Please let me know what you think about this, in my opinion it's a very powerful tool.

Download script file here: <a href="{{'/public/images/871.txt' | prepend: site.baseurl | prepend: site.url }}">871.tcl</a>

{% highlight bash %}
Router(tcl)#source flash:871.tcl
Please enter the hostname of this box: 871_SOHO
Please enter the domain name of this box: HOME.NET
Please enter the username you would like to configure for this system: user
Please enter the password for user: pass
Please enter the enable password: cisco
Please enter the WAN IP address: 1.1.1.1
Please enter the WAN netmask: 255.255.255.0
Please enter the default gateway IP address: 1.1.1.2
Please enter the LAN IP address: 192.168.0.1
Please enter the LAN netmask: 255.255.255.240
Configuring the DHCP service...
Please enter the DNS Server IP address: 8.8.8.8
Please enter the excluded IP address from DHCP assignments - separated by spaces 192.168.0.1 192.168.0.2
Configuring NAT ...
Activating SSH ...
Please enter your timezone [ eg. UTC +2 ]: UTC -5
Done...Your box is waiting for some packets!
{% endhighlight %} 

{% highlight bash %}
871_SOHO#sh run
Building configuration...
!
version 12.4
no service pad
service timestamps debug datetime msec
service timestamps log datetime msec
service password-encryption
!
hostname 871_SOHO
!
boot-start-marker
boot-end-marker
!
logging message-counter syslog
logging buffered 25000
enable secret 5 $1$9xIG$BwuXzBtmLsl6acCURFny31
!
aaa new-model
!
!
aaa authentication login default local
!
!
aaa session-id common
clock timezone UTC -5
!
!
dot11 syslog
ip source-route
!
!
ip dhcp excluded-address 192.168.0.1
ip dhcp excluded-address 192.168.0.2
!
ip dhcp pool DHCP
   network 192.168.0.0 255.255.255.240
   default-router 192.168.0.1
   dns-server 8.8.8.8
!
!
ip cef
no ip domain lookup
ip domain name HOME.NET
no ipv6 cef
ntp server 64.6.144.6
!
multilink bundle-name authenticated
!
!
!
username user privilege 15 password 7 15020A1F17
!
!
!
archive
 log config
  hidekeys
!
!
ip ssh version 2
!
!
!
interface FastEthernet0
 switchport access vlan 55
!
interface FastEthernet1
 switchport access vlan 55
!
interface FastEthernet2
 switchport access vlan 55
!
interface FastEthernet3
 switchport access vlan 55
!
interface FastEthernet4
 ip address 1.1.1.1 255.255.255.0
 ip nat outside
 ip virtual-reassembly
 duplex auto
 speed auto
!
interface Vlan1
 no ip address
 shutdown
!
interface Vlan55
 ip address 192.168.0.1 255.255.255.240
 ip nat inside
 ip virtual-reassembly
!
ip forward-protocol nd
ip route 0.0.0.0 0.0.0.0 1.1.1.2
no ip http server
no ip http secure-server
!
!
ip nat inside source list 1 interface FastEthernet4 overload
!
access-list 1 permit 192.168.0.0 0.0.0.15
!
!
!
!
!
control-plane
!
!
line con 0
 logging synchronous
 no modem enable
line aux 0
line vty 0 4
 length 0
 transport input ssh
!
scheduler max-task-time 5000
end       
{% endhighlight %} 

{% highlight bash %}
ios_config "service password-encryption"
ios_config "no service config"
ios_config "line con 0" "logging synchronous"
puts -nonewline "Please enter the hostname of this box: "
flush stdout
set hostname [ gets stdin ]
puts -nonewline "Please enter the domain name of this box: "
flush stdout
set domain [ gets stdin ]
ios_config "ip domain name $domain"
ios_config "hostname $hostname"
ios_config "logging buffered 25000"
ios_config "ip cef"
ios_config "no ip domain lookup"
ios_config "no ip http server"
ios_config "aaa new-model"
ios_config "aaa authentication login default local"
puts -nonewline "Please enter the username you would like to configure for this system: "
flush stdout
set username [ gets stdin ]
puts -nonewline "Please enter the password for $username: "
flush stdout
set password [ gets stdin ]
ios_config "username $username privilege 15 password $password"
puts -nonewline "Please enter the enable password: "
flush stdout
set enable [ gets stdin ]
ios_config "enable secret $enable"
puts -nonewline "Please enter the WAN IP address: "
flush stdout
set wan_ip [ gets stdin ]
puts -nonewline "Please enter the WAN netmask: "
flush stdout
set wan_netmask [ gets stdin ]
ios_config "interface FastEthernet4" "ip address $wan_ip $wan_netmask"
ios_config "interface FastEthernet4" "no shutdown"
puts -nonewline "Please enter the default gateway IP address: "
flush stdout
set def_gw [ gets stdin ]
ios_config "ip route 0.0.0.0 0.0.0.0 $def_gw"
puts -nonewline "Please enter the LAN IP address: "
flush stdout
set lan_ip [ gets stdin ]
puts -nonewline "Please enter the LAN netmask: "
flush stdout
set lan_netmask [ gets stdin ]
ios_config "interface vlan 1" "shutdown"
ios_config "vlan 55" "name Local"
ios_config "interface vlan 55" "ip address $lan_ip $lan_netmask"
ios_config "interface vlan 55" "no shutdown"
ios_config "interface range FastEthernet 0 - 3" "switchport mode access"
ios_config "interface range FastEthernet 0 - 3" "switchport access vlan 55"
puts "Configuring the DHCP service..."
puts -nonewline "Please enter the DNS Server IP address:"
flush stdout
set dns_ip [ gets stdin ]
puts -nonewline "Please enter the excluded IP address from DHCP assignments - separated by spaces "
flush stdout
set excluded [ gets stdin ]
foreach i $excluded {
        ios_config "ip dhcp excluded-address $i"
}
ios_config "ip dhcp pool DHCP" "network $lan_ip $lan_netmask"
ios_config "ip dhcp pool DHCP" "default-route $lan_ip"
ios_config "ip dhcp pool DHCP" "dns-server $dns_ip"
puts "Configuring NAT ..."
set netmask_oct [split $lan_netmask "."]
foreach j $netmask_oct {
        set wildcard_o [expr {255 - $j}]
        lappend wildcard $wildcard_o
        }
set wildcard_o [split $wildcard " "]
set wildcard_o1 [lindex $wildcard_o 0]
set wildcard_o2 [lindex $wildcard_o 1]
set wildcard_o3 [lindex $wildcard_o 2]
set wildcard_o4 [lindex $wildcard_o 3]
set wildcard_mask "$wildcard_o1.$wildcard_o2.$wildcard_o3.$wildcard_o4"
ios_config "access-list 1 permit $lan_ip $wildcard_mask"
ios_config "ip nat inside source list 1 interface FastEthernet4 overload"
ios_config "interface FastEthernet4" "ip nat outside"
ios_config "interface Vlan 55" "ip nat inside"
puts "Activating SSH ..."
ios_config "line vty 0 4" "transport input ssh"
ios_config "crypto key generate rsa general-keys modulus 1024"
ios_config "ip ssh version 2"
ios_config "ntp server 64.6.144.6"
puts -nonewline {Please enter your timezone [ eg. UTC +2 ]:}
flush stdout
set tzone [ gets stdin ]
ios_config "clock timezone $tzone"
puts "Done...Your box is waiting for some packets!"
{% endhighlight %} 
