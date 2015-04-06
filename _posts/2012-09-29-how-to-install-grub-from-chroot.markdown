---
layout: post
status: publish
published: true
title: How to restore grub using chroot
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 159
wordpress_url: http://www.remote-lab.net/?p=159
date: '2012-09-29 22:53:29 +0000'
date_gmt: '2012-09-29 19:53:29 +0000'
categories:
- Linux
tags: []
comments: []
---
<p>Hello guys,</p>
<p>Below is a small tutorial of how you can install grub by chrooting into the the partition you would like to install the bootloader. I found it useful for XEN to KVM virtual machine migration:</p>
<p><code lang="c[lines-notools]">mount /dev/sda /mnt/<br />
mount -o bind /dev /mnt/dev<br />
chroot /mnt/<br />
mount -t proc none proc<br />
mount sysfs /sys -t sysfs<br />
grub-install -f /dev/sda<br />
grub-mkconfig -o /boot/grub/grub.cfg<br />
reboot</code></p>
