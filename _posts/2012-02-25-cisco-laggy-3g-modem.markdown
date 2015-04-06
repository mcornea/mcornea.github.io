---
layout: post
status: publish
published: true
title: Cisco laggy 3G Modem
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 129
wordpress_url: http://www.remote-lab.net/?p=129
date: '2012-02-25 22:50:10 +0000'
date_gmt: '2012-02-25 19:50:10 +0000'
categories:
- Routing
- IOS
tags: []
comments: []
---
<p>I am writing this post as I've been having these strange issues with my backup 3G Internet link. My primary data provider was down for&nbsp;approximately 2 days so as I presented in this <a href="http://www.remote-lab.net/?p=103">tutorial</a>&nbsp;my router automatically failovers to the backup 3G data. The dynamic failover got in place just fine but after a couple of minutes of using the link it suddenly got stuck. I checked the interface, it was up/up but pings just won't come back - actually I think they didn't even get to leave my router. After a shut/no shut of the interface everything looked back to normal and packets were passing just fine. I investigated further this issue but nothing indicated to a problem, the link was establish and the radio signal was strong.&nbsp;</p>
<p>One solution <a href="http://adminlife.ro/?page_id=10">Adrian</a> came with in hist <a href="http://adminlife.ro/?p=379">post</a>&nbsp;is to track the interface and based on its state to action an EEM script. I adapted a little his script to my needs and here's what I decided to do: I will use the same ip sla to monitor by icmp a core router of my backup data link provider. If within 30 seconds no replies are received then the ip sla goes down and a script which resets the 3G modem is triggered.</p>
<p>One more thing I need to add is a static route to the IP address that I am monitoring in the IP SLA to go through the 3G interface and write the script lines - they are pretty straight forward.&nbsp;</p>
<p>Please notice that the "test cellular" command is enabled after entering the "service internal" command from the config mode.&nbsp;</p>
<p>Please let me know if you have any further questions regarding this setup.</p>
<p><code lang="c[notools]">ip sla 2<br />
 icmp-echo 213.154.121.1 source-interface Dialer2<br />
 timeout 10000<br />
 frequency 10</p>
<p>ip sla schedule 2 life forever start-time now</p>
<p>track 2 ip sla 2 reachability</p>
<p>event manager applet RESET-3G<br />
 event track 2 state down<br />
 action 1.0 cli command "enable"<br />
 action 2.0 cli command "test cellular 0 modem-power-cycle"</code></p>
<p><a href="http://www.remote-lab.net/wp-content/uploads/2012/01/3GBackup.png"><img class="aligncenter size-medium wp-image-104" title="3GBackup" src="http://www.remote-lab.net/wp-content/uploads/2012/01/3GBackup-300x219.png" alt="" width="300" height="219" /></a></p>
