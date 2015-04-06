---
layout: post
status: publish
published: true
title: Cisco certifications - Road to CCNP
author:
  display_name: bogdan
  login: bogdan
  email: bogdan.arvunescu@gmail.com
  url: ''
author_login: bogdan
author_email: bogdan.arvunescu@gmail.com
wordpress_id: 185
wordpress_url: http://www.remote-lab.net/?p=185
date: '2013-02-06 00:06:02 +0000'
date_gmt: '2013-02-05 22:06:02 +0000'
categories:
- Uncategorized
tags: []
comments: []
---
<p><!--[if gte mso 9]><xml></p>
<p><o:OfficeDocumentSettings></p>
<p><o:AllowPNG/></p>
<p></o:OfficeDocumentSettings></p>
<p></xml><![endif]--><span style="font-size: 11.0pt; line-height: 115%; font-family: 'Calibri','sans-serif'; mso-ascii-theme-font: minor-latin; mso-fareast-font-family: Calibri; mso-fareast-theme-font: minor-latin; mso-hansi-theme-font: minor-latin; mso-bidi-font-family: 'Times New Roman'; mso-bidi-theme-font: minor-bidi; mso-ansi-language: EN-US; mso-fareast-language: EN-US; mso-bidi-language: AR-SA;">Hi,</span></p>
<p>My name is Bogdan and I'm a friend with Marius since faculty. I completed my CCNP certification this week and Marius asked me to write an article about what it meant to me this certification and what's my impression now about this course. I'll focus mainly on the pros &amp; cons (I'll try to avoid spoilers here, this is not a 'dump' article about questions/labs from exams):</p>
<p>1.The certification path:</p>
<p>I followed the path ROUTE -&gt; SWITCH -&gt; TSHOOT.</p>
<p>Since it's required to have skill / notions from both switching &amp; routing 'worlds', TSHOOT should be last exam.</p>
<p>2. Are your skills properly tested? Are the exams so difficult?</p>
<p>From my experience, I considered SWITCH as the most difficult and TSHOOT the easiest (since I completed my TSHOOT recently, I'll focus mainly on this exam).</p>
<p>Why TSHOOT was the easiest since it should be considered the last exam in the series and the one that should test all your knowledge in domain?</p>
<p>Cisco has a public link for everyone (you must create an account on <a href="http://cisco.com" target="_blank">cisco.com</a>) regarding the topologies in TSHOOT, also a demo of the interface used:</p>
<p>go to <a href="https://learningnetwork.cisco.com/community/certifications/ccnp/tshoot" target="_blank">https://learningnetwork.cisco.<wbr />com/community/certifications/<wbr />ccnp/tshoot</a>, then choose 'Review the TSHOOT Demo &amp; Tutorial'</p>
<p>Everyone should practice on those diagrams, eventually recreate them in GNS or on real equipment before the exam. In this way you'll definitely review the main objectives of this exam: vlans, trunks, HSRP, DHCP, EIGRP, OSPF, BGP, NAT, IPv6, Tunneling, static routing, ACLs etc.</p>
<p>If you prepare for TSHOOT, you should know what limitations are in CLI - the main one is that you cannot enter in global configuration mode. This could be quite difficult for someone to spot the issue without entering some commands, but in real world you shouldn't test the effects/ solutions on live equipment, rather you should precisely know what's the problem and how it can be solved. OK, this sounds fun &amp; challenging, good job Cisco for creating this type of exam for CCNP! But (here is quite a big problem for me), all the commands that I used to solve the trouble tickets were:</p>
<p>show run<br />
show vlan brief<br />
show ip route / show ipv6 route<br />
show ip eigrp neighbors<br />
show ip ospf neighbors / show ipv6 ospf neighbors<br />
show ip bgp summary<br />
show interface <i>type/mod&nbsp; </i>(since you cannot use 'show interfaces status')<br />
(there also was 'show interfaces trunk', but this was only to double-check some settings)</p>
<p>After all I've learned, resuming the TSHOOT to only 6-7 cli commands is so sad. Only with show run, the problem appears right there in the output! So, after testing the topology again-and-again at home before the exam, I was quite sure what should I expect when I began the real exam. In my opinion, Cisco shouldn't publicly gave the exam topology - OR - have some tickets where you're required to actually configure something. Let's configure a BGP session between 2 routers, with route-maps, prefix-lists, default route redistributed in OSPF, something that requires you to think on before you start entering commands like a robot.</p>
<p>I remember 2 labs from ROUTE &amp; SWITCH, those were challenging and more difficult than the whole TSHOOT exam - one was about redistribution between EIGRP &amp; OSPF and the other one was about choosing some path in STP (actually, preferring some alternate path instead of the default one computed by STP). Instead, in TSHOOT i was like a robot:</p>
<p>-ipconfig from PC<br />
-repeatedly ping nodes in local network / if there was no IP from DHCP server, the solution was even easier<br />
-spot the device on which the issue was<br />
-show ip route / show run</p>
<p>3. After all these, why Cisco? Why not choosing other vendor and read their documentation?</p>
<p>We won't use Cisco equipment forever - Marius knows this situation and I think he's not regretting using another vendors at his work. But the main advantage is that Cisco has a structured way of things in presenting the technologies to you - these are the certifications paths.</p>
<p>After both CCNA &amp; CCNP, I'll definitely forget in the near future their vendor capabilities and features that you're required to learn; but the main topics will still be fresh in my memory. I also might forget the commands or some notions about private vlans for example, but whenever I'll be required to implement this technology, I'll surely remember the main features. Without reading the ROUTE books,&nbsp; I would probably learn about OSPF from another websites or blogs, but never in a structured way.</p>
<p>This is for me the beauty of Cisco exams - you learn a lot of good stuff. It will be always easier to remember some notion you studied a long time ago, rather than learning it for the first time - and usually this is required for you to do in a limited time frame.</p>
<p>4. Should I take the exams or just read the interesting chapters from documentation (please name one person who enjoyed frame-relay topics...)?</p>
<p>There's always a time and a place for everything. I work in an ISP - I know some colleagues with 4-5 years of experience that don't even have their CCNA. But now, after they were confronted with a lot of network issues in these years, they have the skill &amp; knowledge of a CCIE.</p>
<p>Since it was required for me to learn routing &amp; switching notions in detail, I thought that following the certification path will establish the desired goals. In my opinion, more important is the 'hands-on' experience, rather than knowing a lot of stuff without ever implementing them. On the other way, after reaching some level of knowledge will be quite impossible to return and study for an easy exam; let's choose a proper example: my colleagues will always prefer to learn something new instead of reading about OSPF and static routes in CCNA - I think this will be like a punishment for them.</p>
<p>The conclusion is simple, since you're already required to learn something new, why not choosing a certification path? After completing the exam, you'll also gain new notions and also have a world-wide recognized certification in IT world.<span style="color: #888888;"><br style="mso-special-character: line-break;" clear="all" /> </span></p>
<p><!--[if gte mso 9]><xml></p>
<p><w:WordDocument></p>
<p><w:View>Normal</w:View></p>
<p><w:Zoom>0</w:Zoom></p>
<p><w:TrackMoves/></p>
<p><w:TrackFormatting/></p>
<p><w:PunctuationKerning/></p>
<p><w:ValidateAgainstSchemas/></p>
<p><w:SaveIfXMLInvalid>false</w:SaveIfXMLInvalid></p>
<p><w:IgnoreMixedContent>false</w:IgnoreMixedContent></p>
<p><w:AlwaysShowPlaceholderText>false</w:AlwaysShowPlaceholderText></p>
<p><w:DoNotPromoteQF/></p>
<p><w:LidThemeOther>EN-US</w:LidThemeOther></p>
<p><w:LidThemeAsian>X-NONE</w:LidThemeAsian></p>
<p><w:LidThemeComplexScript>X-NONE</w:LidThemeComplexScript></p>
<p><w:Compatibility></p>
<p><w:BreakWrappedTables/></p>
<p><w:SnapToGridInCell/></p>
<p><w:WrapTextWithPunct/></p>
<p><w:UseAsianBreakRules/></p>
<p><w:DontGrowAutofit/></p>
<p><w:SplitPgBreakAndParaMark/></p>
<p><w:EnableOpenTypeKerning/></p>
<p><w:DontFlipMirrorIndents/></p>
<p><w:OverrideTableStyleHps/></p>
<p></w:Compatibility></p>
<p><m:mathPr></p>
<p><m:mathFont m:val="Cambria Math"/></p>
<p><m:brkBin m:val="before"/></p>
<p><m:brkBinSub m:val="--"/></p>
<p><m:smallFrac m:val="off"/></p>
<p><m:dispDef/></p>
<p><m:lMargin m:val="0"/></p>
<p><m:rMargin m:val="0"/></p>
<p><m:defJc m:val="centerGroup"/></p>
<p><m:wrapIndent m:val="1440"/></p>
<p><m:intLim m:val="subSup"/></p>
<p><m:naryLim m:val="undOvr"/></p>
<p></m:mathPr></w:WordDocument></p>
<p></xml><![endif]--><!--[if gte mso 9]><xml></p>
<p><w:LatentStyles DefLockedState="false" DefUnhideWhenUsed="true"   DefSemiHidden="true" DefQFormat="false" DefPriority="99"   LatentStyleCount="267"></p>
<p><w:LsdException Locked="false" Priority="0" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="Normal"/></p>
<p><w:LsdException Locked="false" Priority="9" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="heading 1"/></p>
<p><w:LsdException Locked="false" Priority="9" QFormat="true" Name="heading 2"/></p>
<p><w:LsdException Locked="false" Priority="9" QFormat="true" Name="heading 3"/></p>
<p><w:LsdException Locked="false" Priority="9" QFormat="true" Name="heading 4"/></p>
<p><w:LsdException Locked="false" Priority="9" QFormat="true" Name="heading 5"/></p>
<p><w:LsdException Locked="false" Priority="9" QFormat="true" Name="heading 6"/></p>
<p><w:LsdException Locked="false" Priority="9" QFormat="true" Name="heading 7"/></p>
<p><w:LsdException Locked="false" Priority="9" QFormat="true" Name="heading 8"/></p>
<p><w:LsdException Locked="false" Priority="9" QFormat="true" Name="heading 9"/></p>
<p><w:LsdException Locked="false" Priority="39" Name="toc 1"/></p>
<p><w:LsdException Locked="false" Priority="39" Name="toc 2"/></p>
<p><w:LsdException Locked="false" Priority="39" Name="toc 3"/></p>
<p><w:LsdException Locked="false" Priority="39" Name="toc 4"/></p>
<p><w:LsdException Locked="false" Priority="39" Name="toc 5"/></p>
<p><w:LsdException Locked="false" Priority="39" Name="toc 6"/></p>
<p><w:LsdException Locked="false" Priority="39" Name="toc 7"/></p>
<p><w:LsdException Locked="false" Priority="39" Name="toc 8"/></p>
<p><w:LsdException Locked="false" Priority="39" Name="toc 9"/></p>
<p><w:LsdException Locked="false" Priority="35" QFormat="true" Name="caption"/></p>
<p><w:LsdException Locked="false" Priority="10" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="Title"/></p>
<p><w:LsdException Locked="false" Priority="1" Name="Default Paragraph Font"/></p>
<p><w:LsdException Locked="false" Priority="11" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="Subtitle"/></p>
<p><w:LsdException Locked="false" Priority="22" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="Strong"/></p>
<p><w:LsdException Locked="false" Priority="20" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="Emphasis"/></p>
<p><w:LsdException Locked="false" Priority="59" SemiHidden="false"    UnhideWhenUsed="false" Name="Table Grid"/></p>
<p><w:LsdException Locked="false" UnhideWhenUsed="false" Name="Placeholder Text"/></p>
<p><w:LsdException Locked="false" Priority="1" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="No Spacing"/></p>
<p><w:LsdException Locked="false" Priority="60" SemiHidden="false"    UnhideWhenUsed="false" Name="Light Shading"/></p>
<p><w:LsdException Locked="false" Priority="61" SemiHidden="false"    UnhideWhenUsed="false" Name="Light List"/></p>
<p><w:LsdException Locked="false" Priority="62" SemiHidden="false"    UnhideWhenUsed="false" Name="Light Grid"/></p>
<p><w:LsdException Locked="false" Priority="63" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Shading 1"/></p>
<p><w:LsdException Locked="false" Priority="64" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Shading 2"/></p>
<p><w:LsdException Locked="false" Priority="65" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium List 1"/></p>
<p><w:LsdException Locked="false" Priority="66" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium List 2"/></p>
<p><w:LsdException Locked="false" Priority="67" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 1"/></p>
<p><w:LsdException Locked="false" Priority="68" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 2"/></p>
<p><w:LsdException Locked="false" Priority="69" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 3"/></p>
<p><w:LsdException Locked="false" Priority="70" SemiHidden="false"    UnhideWhenUsed="false" Name="Dark List"/></p>
<p><w:LsdException Locked="false" Priority="71" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful Shading"/></p>
<p><w:LsdException Locked="false" Priority="72" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful List"/></p>
<p><w:LsdException Locked="false" Priority="73" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful Grid"/></p>
<p><w:LsdException Locked="false" Priority="60" SemiHidden="false"    UnhideWhenUsed="false" Name="Light Shading Accent 1"/></p>
<p><w:LsdException Locked="false" Priority="61" SemiHidden="false"    UnhideWhenUsed="false" Name="Light List Accent 1"/></p>
<p><w:LsdException Locked="false" Priority="62" SemiHidden="false"    UnhideWhenUsed="false" Name="Light Grid Accent 1"/></p>
<p><w:LsdException Locked="false" Priority="63" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Shading 1 Accent 1"/></p>
<p><w:LsdException Locked="false" Priority="64" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Shading 2 Accent 1"/></p>
<p><w:LsdException Locked="false" Priority="65" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium List 1 Accent 1"/></p>
<p><w:LsdException Locked="false" UnhideWhenUsed="false" Name="Revision"/></p>
<p><w:LsdException Locked="false" Priority="34" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="List Paragraph"/></p>
<p><w:LsdException Locked="false" Priority="29" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="Quote"/></p>
<p><w:LsdException Locked="false" Priority="30" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="Intense Quote"/></p>
<p><w:LsdException Locked="false" Priority="66" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium List 2 Accent 1"/></p>
<p><w:LsdException Locked="false" Priority="67" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 1 Accent 1"/></p>
<p><w:LsdException Locked="false" Priority="68" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 2 Accent 1"/></p>
<p><w:LsdException Locked="false" Priority="69" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 3 Accent 1"/></p>
<p><w:LsdException Locked="false" Priority="70" SemiHidden="false"    UnhideWhenUsed="false" Name="Dark List Accent 1"/></p>
<p><w:LsdException Locked="false" Priority="71" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful Shading Accent 1"/></p>
<p><w:LsdException Locked="false" Priority="72" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful List Accent 1"/></p>
<p><w:LsdException Locked="false" Priority="73" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful Grid Accent 1"/></p>
<p><w:LsdException Locked="false" Priority="60" SemiHidden="false"    UnhideWhenUsed="false" Name="Light Shading Accent 2"/></p>
<p><w:LsdException Locked="false" Priority="61" SemiHidden="false"    UnhideWhenUsed="false" Name="Light List Accent 2"/></p>
<p><w:LsdException Locked="false" Priority="62" SemiHidden="false"    UnhideWhenUsed="false" Name="Light Grid Accent 2"/></p>
<p><w:LsdException Locked="false" Priority="63" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Shading 1 Accent 2"/></p>
<p><w:LsdException Locked="false" Priority="64" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Shading 2 Accent 2"/></p>
<p><w:LsdException Locked="false" Priority="65" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium List 1 Accent 2"/></p>
<p><w:LsdException Locked="false" Priority="66" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium List 2 Accent 2"/></p>
<p><w:LsdException Locked="false" Priority="67" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 1 Accent 2"/></p>
<p><w:LsdException Locked="false" Priority="68" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 2 Accent 2"/></p>
<p><w:LsdException Locked="false" Priority="69" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 3 Accent 2"/></p>
<p><w:LsdException Locked="false" Priority="70" SemiHidden="false"    UnhideWhenUsed="false" Name="Dark List Accent 2"/></p>
<p><w:LsdException Locked="false" Priority="71" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful Shading Accent 2"/></p>
<p><w:LsdException Locked="false" Priority="72" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful List Accent 2"/></p>
<p><w:LsdException Locked="false" Priority="73" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful Grid Accent 2"/></p>
<p><w:LsdException Locked="false" Priority="60" SemiHidden="false"    UnhideWhenUsed="false" Name="Light Shading Accent 3"/></p>
<p><w:LsdException Locked="false" Priority="61" SemiHidden="false"    UnhideWhenUsed="false" Name="Light List Accent 3"/></p>
<p><w:LsdException Locked="false" Priority="62" SemiHidden="false"    UnhideWhenUsed="false" Name="Light Grid Accent 3"/></p>
<p><w:LsdException Locked="false" Priority="63" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Shading 1 Accent 3"/></p>
<p><w:LsdException Locked="false" Priority="64" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Shading 2 Accent 3"/></p>
<p><w:LsdException Locked="false" Priority="65" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium List 1 Accent 3"/></p>
<p><w:LsdException Locked="false" Priority="66" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium List 2 Accent 3"/></p>
<p><w:LsdException Locked="false" Priority="67" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 1 Accent 3"/></p>
<p><w:LsdException Locked="false" Priority="68" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 2 Accent 3"/></p>
<p><w:LsdException Locked="false" Priority="69" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 3 Accent 3"/></p>
<p><w:LsdException Locked="false" Priority="70" SemiHidden="false"    UnhideWhenUsed="false" Name="Dark List Accent 3"/></p>
<p><w:LsdException Locked="false" Priority="71" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful Shading Accent 3"/></p>
<p><w:LsdException Locked="false" Priority="72" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful List Accent 3"/></p>
<p><w:LsdException Locked="false" Priority="73" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful Grid Accent 3"/></p>
<p><w:LsdException Locked="false" Priority="60" SemiHidden="false"    UnhideWhenUsed="false" Name="Light Shading Accent 4"/></p>
<p><w:LsdException Locked="false" Priority="61" SemiHidden="false"    UnhideWhenUsed="false" Name="Light List Accent 4"/></p>
<p><w:LsdException Locked="false" Priority="62" SemiHidden="false"    UnhideWhenUsed="false" Name="Light Grid Accent 4"/></p>
<p><w:LsdException Locked="false" Priority="63" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Shading 1 Accent 4"/></p>
<p><w:LsdException Locked="false" Priority="64" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Shading 2 Accent 4"/></p>
<p><w:LsdException Locked="false" Priority="65" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium List 1 Accent 4"/></p>
<p><w:LsdException Locked="false" Priority="66" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium List 2 Accent 4"/></p>
<p><w:LsdException Locked="false" Priority="67" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 1 Accent 4"/></p>
<p><w:LsdException Locked="false" Priority="68" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 2 Accent 4"/></p>
<p><w:LsdException Locked="false" Priority="69" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 3 Accent 4"/></p>
<p><w:LsdException Locked="false" Priority="70" SemiHidden="false"    UnhideWhenUsed="false" Name="Dark List Accent 4"/></p>
<p><w:LsdException Locked="false" Priority="71" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful Shading Accent 4"/></p>
<p><w:LsdException Locked="false" Priority="72" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful List Accent 4"/></p>
<p><w:LsdException Locked="false" Priority="73" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful Grid Accent 4"/></p>
<p><w:LsdException Locked="false" Priority="60" SemiHidden="false"    UnhideWhenUsed="false" Name="Light Shading Accent 5"/></p>
<p><w:LsdException Locked="false" Priority="61" SemiHidden="false"    UnhideWhenUsed="false" Name="Light List Accent 5"/></p>
<p><w:LsdException Locked="false" Priority="62" SemiHidden="false"    UnhideWhenUsed="false" Name="Light Grid Accent 5"/></p>
<p><w:LsdException Locked="false" Priority="63" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Shading 1 Accent 5"/></p>
<p><w:LsdException Locked="false" Priority="64" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Shading 2 Accent 5"/></p>
<p><w:LsdException Locked="false" Priority="65" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium List 1 Accent 5"/></p>
<p><w:LsdException Locked="false" Priority="66" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium List 2 Accent 5"/></p>
<p><w:LsdException Locked="false" Priority="67" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 1 Accent 5"/></p>
<p><w:LsdException Locked="false" Priority="68" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 2 Accent 5"/></p>
<p><w:LsdException Locked="false" Priority="69" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 3 Accent 5"/></p>
<p><w:LsdException Locked="false" Priority="70" SemiHidden="false"    UnhideWhenUsed="false" Name="Dark List Accent 5"/></p>
<p><w:LsdException Locked="false" Priority="71" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful Shading Accent 5"/></p>
<p><w:LsdException Locked="false" Priority="72" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful List Accent 5"/></p>
<p><w:LsdException Locked="false" Priority="73" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful Grid Accent 5"/></p>
<p><w:LsdException Locked="false" Priority="60" SemiHidden="false"    UnhideWhenUsed="false" Name="Light Shading Accent 6"/></p>
<p><w:LsdException Locked="false" Priority="61" SemiHidden="false"    UnhideWhenUsed="false" Name="Light List Accent 6"/></p>
<p><w:LsdException Locked="false" Priority="62" SemiHidden="false"    UnhideWhenUsed="false" Name="Light Grid Accent 6"/></p>
<p><w:LsdException Locked="false" Priority="63" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Shading 1 Accent 6"/></p>
<p><w:LsdException Locked="false" Priority="64" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Shading 2 Accent 6"/></p>
<p><w:LsdException Locked="false" Priority="65" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium List 1 Accent 6"/></p>
<p><w:LsdException Locked="false" Priority="66" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium List 2 Accent 6"/></p>
<p><w:LsdException Locked="false" Priority="67" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 1 Accent 6"/></p>
<p><w:LsdException Locked="false" Priority="68" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 2 Accent 6"/></p>
<p><w:LsdException Locked="false" Priority="69" SemiHidden="false"    UnhideWhenUsed="false" Name="Medium Grid 3 Accent 6"/></p>
<p><w:LsdException Locked="false" Priority="70" SemiHidden="false"    UnhideWhenUsed="false" Name="Dark List Accent 6"/></p>
<p><w:LsdException Locked="false" Priority="71" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful Shading Accent 6"/></p>
<p><w:LsdException Locked="false" Priority="72" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful List Accent 6"/></p>
<p><w:LsdException Locked="false" Priority="73" SemiHidden="false"    UnhideWhenUsed="false" Name="Colorful Grid Accent 6"/></p>
<p><w:LsdException Locked="false" Priority="19" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="Subtle Emphasis"/></p>
<p><w:LsdException Locked="false" Priority="21" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="Intense Emphasis"/></p>
<p><w:LsdException Locked="false" Priority="31" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="Subtle Reference"/></p>
<p><w:LsdException Locked="false" Priority="32" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="Intense Reference"/></p>
<p><w:LsdException Locked="false" Priority="33" SemiHidden="false"    UnhideWhenUsed="false" QFormat="true" Name="Book Title"/></p>
<p><w:LsdException Locked="false" Priority="37" Name="Bibliography"/></p>
<p><w:LsdException Locked="false" Priority="39" QFormat="true" Name="TOC Heading"/></p>
<p></w:LatentStyles></p>
<p></xml><![endif]--><!--[if gte mso 10]></p>
<style>
 /* Style Definitions */</p>
<p>table.MsoNormalTable</p>
<p>{mso-style-name:"Table Normal";</p>
<p>mso-tstyle-rowband-size:0;</p>
<p>mso-tstyle-colband-size:0;</p>
<p>mso-style-noshow:yes;</p>
<p>mso-style-priority:99;</p>
<p>mso-style-parent:"";</p>
<p>mso-padding-alt:0in 5.4pt 0in 5.4pt;</p>
<p>mso-para-margin-top:0in;</p>
<p>mso-para-margin-right:0in;</p>
<p>mso-para-margin-bottom:10.0pt;</p>
<p>mso-para-margin-left:0in;</p>
<p>line-height:115%;</p>
<p>mso-pagination:widow-orphan;</p>
<p>font-size:11.0pt;</p>
<p>font-family:"Calibri","sans-serif";</p>
<p>mso-ascii-font-family:Calibri;</p>
<p>mso-ascii-theme-font:minor-latin;</p>
<p>mso-hansi-font-family:Calibri;</p>
<p>mso-hansi-theme-font:minor-latin;}</p>
</style>
<p><![endif]--></p>
