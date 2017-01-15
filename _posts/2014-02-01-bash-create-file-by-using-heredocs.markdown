---
layout: post
comments: True
title: Bash create file by using heredocs
date: '2014-02-01 22:51:47 +0000'
categories:
- Linux
- Automation
permalink: bash-heredocs
---
By using the heredocs format you instruct the shell to read input from the current source until a line containing only a certain word is seen. I find this useful for quickly writing or copying script files. You may find below an example of how you can use this kind of redirection:

___

{% highlight bash %}
marius@remoteur:~>>> cat > script << EOF
> echo "This is the script"
> EOF
marius@remoteur:~>>> bash script
This is the script
{% endhighlight %} 

