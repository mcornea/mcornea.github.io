---
layout: post
status: publish
published: true
title: KVM Installation on Ubuntu 12.10
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 177
wordpress_url: http://www.remote-lab.net/?p=177
date: '2013-04-19 16:35:02 +0000'
date_gmt: '2013-04-19 14:35:02 +0000'
categories:
- Linux
- Virtualization
tags: []
comments: []
---

<p>In this post I'd like to do an introduction on how you can install KVM and the tools that allow you to easily run VMs.</p>
<p>KVM - Kernel Virtual Machine is a Linux kernel module that enables the user space programs to use the hardware virtualization capabilities of Intel or AMD processors.</p>
<p>QEMU is a machine emulator that allows you to run instructions for specific architecture on your x86 CPU. It is also a virtualizer that executes the guest VM code directly on the hypervisor CPU, which is our use case providing almost native performance for the guest VMs.</p>
<p>The libvirt library provides an interface used for the KVM instances management.</p>
<p>bridge-utils allow you to bridge multiple ethernet devices and it will help you provide connectivity to your VM instances.</p>
<p>virtinst provides cli tools that allow you to create VMs</p>
<p>virt-manager is a graphical tool used for the VM management</p>
<p><code lang="c[notools]">marius@remoteur:~\> sudo aptitude install qemu-kvm libvirt-bin bridge-utils virtinst virt-manager</code></p>
<p>The next step is to add our username to the libvirtd group:</p>
<p><code lang="c[notools]">marius@remoteur:~\> sudo adduser `id -un` libvirtd</code></p>
<p>At this time we should be able to start creating an instance. Let's prepare the storage and the network our VM will use.</p>
<p>For the storage we will create an emtpy image file of 5GiB size:</p>
<p><code lang="c[notools]">marius@remoteur:~\> dd if=/dev/zero of=vm.storage bs=1024 count=0 seek=$[1024*1024*5]</code></p>
<p>We will create a bridge called br0 and add eth0 wich is my laptop wired network interface to it. When created the VM should have an interface called vnet0 that is part of this bridge group:</p>
<p><code lang="email[notools]">marius@remoteur:~\> sudo brctl add br0<br />
marius@remoteur:~\> sudo brctl addif br0 eth0<br />
marius@remoteur:~\> brctl show<br />
bridge name      bridge id        STP enabled      interfaces<br />
br0           8000.c82a141ab88e        no          eth0<br />
</code></p>
<p>Now let's define our VM instance.</p>
<p><code lang="email[notools]">marius@remoteur:~\> sudo virt-install --name vm --disk path=/home/marius/vm.storage,bus=virtio,cache=none --network bridge=br0,model=virtio --location=ftp://ftp.lug.ro/debian/dists/wheezy/main/installer-amd64/ --cpuset=auto --vcpu=1 --ram=1024 --extra-args="priority=low"</code></p>
<p>After this our install should be starting. You can use virt-manager to manage all the machines that are running on your local or remote hypervisor. It also allows you to access their consoles and adjust different parameters in the VM definition.</p>
<p><a href="https://www.remote-lab.net/wp-content/uploads/2013/01/virt-manager.png"><img class="aligncenter size-medium wp-image-147" title="virt-manager" alt="" src="https://www.remote-lab.net/wp-content/uploads/2013/01/virt-manager.png" width="500" height="465" /></a></p>
