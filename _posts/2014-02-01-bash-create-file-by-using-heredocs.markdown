---
layout: post
status: publish
published: true
title: Bash create file by using heredocs
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 210
wordpress_url: https://remote-lab.net/?p=210
date: '2014-02-01 22:51:47 +0000'
date_gmt: '2014-02-01 20:51:47 +0000'
categories:
- Linux
tags: []
comments: []
---
<p>By using the heredocs format you instruct the shell to read input from the current source until a line containing only a certain word is seen. I find this useful for quickly writing or copying script files. You may find below an example of how you can use this kind of redirection:</p>
<p><code lang="c[notools]">marius@remoteur:~>>> cat > script << EOF<br />
> echo "This is the script"<br />
> EOF<br />
marius@remoteur:~>>> bash script<br />
This is the script</code></p>
