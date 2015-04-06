---
layout: post
status: publish
published: true
title: Append and prepend using sed
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 152
wordpress_url: http://www.remote-lab.net/?p=152
date: '2012-08-25 20:46:33 +0000'
date_gmt: '2012-08-25 17:46:33 +0000'
categories:
- Linux
tags: []
comments: []
---
<p>Hello guys,</p>
<p>Below is a small example that uses sed to create null routes based on the domain name. It can be useful when the domain name resolves to multiple IP addresses :&nbsp;</p>
<p><code lang="c[notools]">host facebook.com| awk {'print $4'} | sed -n '/^[1-9].*/p' | sed 's/^/ip route /;s/$/ 255.255.255.255 null0/'</p>
<p>The result:<br />
ip route 69.171.234.21 255.255.255.255 null0<br />
ip route 69.171.237.16 255.255.255.255 null0<br />
ip route 69.171.247.21 255.255.255.255 null0<br />
ip route 66.220.149.88 255.255.255.255 null0<br />
ip route 66.220.152.16 255.255.255.255 null0<br />
ip route 66.220.158.70 255.255.255.255 null0<br />
</code></p>
<p>&nbsp;</p>
