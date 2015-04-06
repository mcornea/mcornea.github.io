---
layout: post
title: How to restore grub using chroot
wordpress_url: http://www.remote-lab.net/?p=159
date: '2012-09-29 22:53:29 +0000'
categories:
- Linux
permalink: how-to-install-grub-from-chroot
---
Below is a small tutorial of how you can install grub by chrooting into the the partition you would like to install the bootloader. 

I found it useful for XEN to KVM virtual machine migration:

___

{% highlight bash %}
mount /dev/sda /mnt/
mount -o bind /dev /mnt/dev
chroot /mnt/
mount -t proc none proc
mount sysfs /sys -t sysfs
grub-install -f /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
reboot
{% endhighlight %} 
