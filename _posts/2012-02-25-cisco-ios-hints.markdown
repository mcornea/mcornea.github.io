---
layout: post
status: publish
published: true
title: Cisco IOS Hints
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 130
wordpress_url: http://www.remote-lab.net/?p=130
date: '2012-02-25 22:59:54 +0000'
date_gmt: '2012-02-25 19:59:54 +0000'
categories:
- IOS
tags: []
comments: []
---
<p>Hey guys,</p>
<p>A nice tip I found about today is how to get rid of the space scrolling when showing a config, log, etc. This may be useful when you want to copy the configuration to a text editor and you don't want to press space so many times to scroll until the end.&nbsp;</p>
<p><code lang="c[notools]">Router#terminal length 0</code></p>
<p>and to have it back to the initial settings:</p>
<p><code lang="c[notools]">Router#terminal length 24</code></p>
