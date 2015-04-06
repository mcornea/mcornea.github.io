---
layout: post
status: publish
published: true
title: Cisco 881G - Configuring Dynamic Failover On a Backup 3G Cellular Link
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 103
wordpress_url: http://www.remote-lab.net/?p=103
date: '2012-01-09 21:33:38 +0000'
date_gmt: '2012-01-09 18:33:38 +0000'
categories:
- Routing
tags: []
comments:
- id: 70
  author: Adrian Brunhuber
  author_email: adi@adminlife.ro
  author_url: http://adminlife.ro/
  date: '2012-01-17 11:17:04 +0000'
  date_gmt: '2012-01-17 08:17:04 +0000'
  content: "Salut,frumos articol,intr-adevar asa se configureaza o reduntanta intre
    2 provideri, acest tutorial se poate aplica la orice fel de conexiune. Singura
    problema ce o vad ca in realitatea track-ul respectiv va cadea si se va ridica
    de multe ori pe zi.La fiecare ping primit cu o latenta mai mare de 200ms va trece
    pe 3G,nepastrand translatiile nat,acest lucru fiind simtit de userii din retea.Dupa
    care imediat va trece inapoi.Eu as face o mica modificare.Am pune track-ul sa
    cada daca nu primeste ping sau latentele sunt mari mai mult de x secunde,nu la
    primul ping pierdut sa o ia razna.Arunca o privire peste optiunea ce tine de SLA
    (delay down si delay up).Totodata in practica as face un script care sa imi monitorizeze
    conexiunea de vodafone,3g-urile fiind cunoscute ca se \"agatza\",iar daca fail-over-ul
    se face pe o conexiune moarta nu mai e failover.\r\nEventual da-mi mail sa iti
    dau un exemplu de script pentru 3g.\r\nO zi superba,\r\nMatache Retelaru`"
- id: 71
  author: marius
  author_email: marius@remote-lab.net
  author_url: ''
  date: '2012-01-18 20:31:42 +0000'
  date_gmt: '2012-01-18 17:31:42 +0000'
  content: "Salut Adrian,\r\n\r\nTi-am trimis un mail referitor la script-ul pentru
    modem-ul 3G. Intr-adevar am vazut ca link-ul 3G nu prea e stabil. O sa citesc
    si documentatia pentru SLA-uri sa vad cum e cu delay-urile si cum se pot optimiza.
    \r\nAm vazut ca ai inceput o serie de tutoriale pentru ASA-uri, chiar imi doream
    asa ceva si o sa vin cu un feedback - tutorialele de basic config sunt foarte
    bine venite pentru beginneri ca mine :)\r\n\r\nMarius"
- id: 91
  author: Cisco laggy 3G Modem | Remote-Lab.net &#8211; Free Cisco and Linux Training
    for Networking Professionals
  author_email: ''
  author_url: http://www.remote-lab.net/?p=129
  date: '2012-02-25 22:50:15 +0000'
  date_gmt: '2012-02-25 19:50:15 +0000'
  content: "[...] Internet link. My primary data provider was down for&nbsp;approximately
    so as I presented in this tutorial&nbsp;my router automatically failovers to the
    backup 3G data. The dynamic failover got in place [...]"
- id: 7457
  author: jimwiltshire
  author_email: jim_wiltshire@hotmail.com
  author_url: ''
  date: '2013-07-31 15:53:17 +0000'
  date_gmt: '2013-07-31 13:53:17 +0000'
  content: "Hi Marius,\r\n\r\n    I was hoping you would be able to lend me your expertise
    with configuring the failover segment for the environment I have our 881G set
    up on? I was able to configure the dialer portion with no issues but am having
    issues with the failover. Drop me a message. Thanks Marius."
- id: 7591
  author: marius
  author_email: marius@remote-lab.net
  author_url: http://www.remote-lab.net/
  date: '2013-08-05 23:06:19 +0000'
  date_gmt: '2013-08-05 21:06:19 +0000'
  content: Sure, let me know what is your setup is and we'll figure it out. My email
    address is marius@remote-lab.net
- id: 17615
  author: Mihail
  author_email: leancamihail@gmail.com
  author_url: ''
  date: '2015-01-12 10:32:20 +0000'
  date_gmt: '2015-01-12 08:32:20 +0000'
  content: "Merci mult pentru config de mai sus. Dar totusi di outside nu pot face
    ping la ambele WAN IP obtinute (Fa4 si Cell 0). Cum pot sa obtin acest rezultat!!!\r\nMultumesc!!!"
---
<p>Hello guys,</p>
<p>My latest lab toy is a Cisco 881G which integrates a cellular 3G interface used a secondary data link. In the following article I will present the existing scenario and what needs to be acomplished.</p>
<p><a href="http://www.remote-lab.net/wp-content/uploads/2012/01/3GBackup.png"><img class="aligncenter size-large wp-image-104" title="3GBackup" src="http://www.remote-lab.net/wp-content/uploads/2012/01/3GBackup-1024x748.png" alt="" width="550" height="401" /></a></p>
<p>What's the scenario? We have a small LAN using private IP addresses which need to be translated on both the external links. The primary link is an Ethernet drop so it's very unlikely to change its state to down so that the router can detect the link failure and make the changes in its RIB. In order to detect a failure on the primary link we have to implement a mechanism which takes an external point as reference and measures different parameters.  For instance in my scenario we will send ICMP packets to an external IP address - 8.8.8.8, the Google public DNS server, and measure the RTT. As long as the RTT value is within certain limits the default route will be set on the primary link.<br />
If it exceeds that value then the default route set on the primary link will be dropped out of the routing table and a secondary default route - the one from the backup link, which has a higher administrative distance will take its place.</p>
<p>We have to make sure that the ICMP packets sent for checking the RTT will be sent only from the primary link. Otherwise when the backup link becomes active it checks the RTT and gets a proper value and it install the primary default route thus resulting in a loop of default routes.</p>
<p>Enough with the words, now let's get to the config lines:</p>
<p>First of all, let's configure the cellular interface so that it can connect to the 3G network:</p>
<h5>Configuring the cellular interface</h5>
<p>We must ensure that the SIM card is not locked with a PIN code - Cellular 0 is the interface:</p>
<pre>Router#cellular 0 gsm sim unlock <em>PIN</em></pre>
<p><em></em>   We need to create a gsm profile with the settings associated with your data account:</p>
<pre>Router#cellular 0 gsm profile create 1 internet ipv4 pap cisco cisco</pre>
<p>"1" - Profile number<br />
"internet" - Access Point Name<br />
"ipv4" - Packet Data Protocol<br />
"pap" - Authentication type<br />
"cisco" - Username<br />
"cisco" - Password</p>
<p>Next we need to create a chat script which sends commands to the 3G modem to connect to the remote system. The chat script is called CellScript with a timeout value of 30 seconds. Please note that the script may be different depending on the carrier.</p>
<pre>Router(config)#chat-script CellScript "" "ATDT*98*<strong>1</strong>#" TIMEOUT 30 "CONNECT"</pre>
<p>"1" - represents the number of the gsm profile created above</p>
<p>Associate the created chat script to the 3G modem line, for the 881G is line 3:</p>
<pre>Router(config)#line 3
Router(config-line)#script dialer CellScript</pre>
<p>Next we should create a Dialer interface:</p>
<pre>Router(config)#interface Dialer2
Router(config-if)#ip address negotiated
Router(config-if)#ip virtual-reassembly in
Router(config-if)#encapsulation ppp
Router(config-if)#dialer pool 2
Router(config-if)#dialer idle-timeout 0
Router(config-if)#dialer string CellScript
Router(config-if)#dialer persistent
Router(config-if)#dialer-group 2
Router(config-if)#ppp authentication pap callin
Router(config-if)#ppp pap password 7 045F1E0B0238
Router(config-if)#ppp ipcp dns request
Router(config-if)#no cdp enable</pre>
<p>The next step required is to configure the Cellular interface</p>
<pre>Router(config)#interface Cellular0
Router(config-if)#ip address negotiated
Router(config-if)#ip virtual-reassembly in
Router(config-if)#encapsulation ppp
Router(config-if)#dialer in-band
Router(config-if)#dialer pool-member 2
Router(config-if)#dialer idle-timeout 0
Router(config-if)#dialer-group 2
Router(config-if)#async mode interactive</pre>
<p>Create a dialer list which allows IP packets for the interfaces in dialer group 2:</p>
<pre> Router(config)#dialer-list 2 protocol ip permit</pre>
<p>Add a default route and you should have a working 3G data link:</p>
<pre>Router(config)#ip route 0.0.0.0 0.0.0.0 Dialer2</pre>
<h5> Configuring NAT with multiple pools</h5>
<p>OK, now that we have set the 3G backup data link I presume that you know how to configure the primary Ethernet link so we can pass to the next section: how to get Network Address Translation work on both interfaces.</p>
<p>First we need to set an access list which identifies our private addresses - in my case: 10.0.0.0/28</p>
<pre>Router(config)#access-list 10 permit 10.0.0.0 0.0.0.15</pre>
<p>Next we'll create route maps which identify both the private ip addresses by the ACL created above and the external interface:</p>
<pre>Router(config)#route-map PRIMARY permit 1 -&gt; 1 is a sequence number in the route map function
Router(config-route-map) #match ip address 10 -&gt; matches all IP traffic identified by ACL 10
Router(config-route-map)#match interface FastEthernet4</pre>
<pre>Router(config)#route-map BACKUP permit 1
Router(config-route-map) #match ip address 10
Router(config-route-map)#match interface Dialer2</pre>
<p>Last we have to set the NAT direction on the interfaces we are interested in:</p>
<pre>Router(config)#interface Vlan 55
Router(config-if)#ip nat inside</pre>
<pre>Router(config)#interface FastEthernet4
Router(config-if)#ip nat outside</pre>
<pre>Router(config)#interface Dialer2
Router(config-if)#ip nat outside</pre>
<p>Now that we have the route maps set we need the create the NAT translation rules. On both of the interfaces I will set NAT overload as I have a single IP address assigned by the ISP:</p>
<pre>Router(config)# ip nat inside source route-map PRIMARY interface FastEthernet4 overload
Router(config)# ip nat inside source route-map BACKUP interface Dialer2 overload</pre>
<h5>Configuring dynamic failover with Cisco IP SLAs</h5>
<p>The address translation is complete too. In the next section we will implement the dynamic failover mechanism using Cisco IP Service Level Agreements ( SLAs ). The Cisco IOS IP SLA is a powerful tool which can help improve your networks services. It allows you to monitor traffic parameters like round-trip delay, one-way delay, one-way jitter, packet loss, TCP connection time, DNS lookup time and many others. Let's get to the config line and show how we should implement the SLAs.</p>
<pre>Router(config)#ip sla 1
Router(config)#icmp-echo 8.8.8.8 interface FastEthernet 4 -&gt; pings 8.8.8.8 with the source address of Fa4
Router(config)#timeout 150 -&gt; the amount of time in milliseconds the IP SLA operation waits a response from its request packet
Router(config)#frequency  2 -&gt; the amount of time in seconds at which the IP SLA operation performs
Router(config)#ip sla schedule 1 life forever start-time now</pre>
<p>To review the above config lines: The routers pings 8.8.8.8 with  ICMP REQUEST packets having the source address of Fa4 and if an ICMP REPLY packet isn't received within 150ms the IP SLA operation is considered to have failed. This process is repeated once every 2 seconds.<br />
We set this process to start now and occur forever.</p>
<p>Now what happens if the IP SLA operation fails? In the following lines we'll tell the router what to do in that case:</p>
<p>We need to create a track object to monitor the status of the IP SLA we have created:</p>
<pre>Router(config)#track 1 ip sla 1 reachability</pre>
<p>The track object informs the static route if a state change of the IP SLA occurs.</p>
<p>Next we will associate the primary default route with the tracker.</p>
<pre>Router(config)#ip route 0.0.0.0 0.0.0.0 FastEthernet4 track 1</pre>
<p>and assign a higher administrative distance to the secondary default route:</p>
<pre>Router(config)#ip route 0.0.0.0 0.0.0.0 Dialer2 10</pre>
<p>And that should be all. You should now have a dynamic failover route without having to enable dynamic routing protocols.</p>
<p>Pretty cool :)</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
