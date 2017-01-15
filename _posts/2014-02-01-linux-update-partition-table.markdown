---
layout: post
comments: True
title: Linux reload partition table
date: '2014-02-01 22:14:56 +0000'
categories:
- Linux
permalink: linux-update-partition-table
---
Below are the steps you can use in order to be able to create a filesystem on a newly created partition (vda3 in our example) without reboot:

___

{% highlight bash %}
root@lfs:~# fdisk -l
Disk /dev/vda: 16.1 GB, 16106127360 bytes
16 heads, 63 sectors/track, 31207 cylinders, total 31457280 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0003d859
   Device Boot      Start         End      Blocks   Id  System
/dev/vda1            2048     1953791      975872   82  Linux swap / Solaris
/dev/vda2   *     1953792    11718655     4882432   83  Linux
/dev/vda3        11718656    31457279     9869312   83  Linux
root@lfs:~# ls /dev/vda
vda   vda1  vda2
root@lfs:~# aptitude install parted
root@lfs:~# partprobe
root@lfs:~# ls /dev/vda
vda   vda1  vda2  vda3
root@lfs:~# mkfs.ext4 /dev/vda3
{% endhighlight %} 

