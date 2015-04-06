---
layout: post
title: KVM Installation on Ubuntu 12.10
date: '2013-04-19 16:35:02 +0000'
categories:
- Linux
- Virtualization
permalink: kvm-installation-on-ubuntu-12-10
---

In this post I'd like to do an introduction on how you can install KVM and the tools that allow you to easily run VMs.
KVM - Kernel Virtual Machine is a Linux kernel module that enables the user space programs to use the hardware virtualization capabilities of Intel or AMD processors.

___

QEMU is a machine emulator that allows you to run instructions for specific architecture on your x86 CPU. It is also a virtualizer that executes the guest VM code directly on the hypervisor CPU, which is our use case providing almost native performance for the guest VMs.

The libvirt library provides an interface used for the KVM instances management.

bridge-utils allow you to bridge multiple ethernet devices and it will help you provide connectivity to your VM instances.

virtinst provides cli tools that allow you to create VMs

virt-manager is a graphical tool used for the VM management

{% highlight bash %}
marius@remoteur:~\> sudo aptitude install qemu-kvm libvirt-bin bridge-utils virtinst virt-manager
{% endhighlight %} 

The next step is to add our username to the libvirtd group:
{% highlight bash %}
marius@remoteur:~\> sudo adduser `id -un` libvirtd
{% endhighlight %} 

At this time we should be able to start creating an instance. Let's prepare the storage and the network our VM will use.
For the storage we will create an emtpy image file of 5GiB size:
{% highlight bash %}
marius@remoteur:~\> dd if=/dev/zero of=vm.storage bs=1024 count=0 seek=$[1024*1024*5]
{% endhighlight %} 

We will create a bridge called br0 and add eth0 wich is my laptop wired network interface to it. When created the VM should have an interface called vnet0 that is part of this bridge group:
{% highlight bash %}
marius@remoteur:~\> sudo brctl add br0
marius@remoteur:~\> sudo brctl addif br0 eth0
marius@remoteur:~\> brctl show
bridge name      bridge id        STP enabled      interfaces
br0           8000.c82a141ab88e        no          eth0
{% endhighlight %} 

Now let's define our VM instance.
{% highlight bash %}
marius@remoteur:~\> sudo virt-install --name vm --disk path=/home/marius/vm.storage,bus=virtio,cache=none --network bridge=br0,model=virtio --location=ftp://ftp.lug.ro/debian/dists/wheezy/main/installer-amd64/ --cpuset=auto --vcpu=1 --ram=1024 --extra-args="priority=low"
{% endhighlight %} 


After this our install should be starting. You can use virt-manager to manage all the machines that are running on your local or remote hypervisor. It also allows you to access their consoles and adjust different parameters in the VM definition.
<a href="{{'assets/static/virt-manager.png' | prepend: site.baseurl | prepend: site.url }}"><img class="aligncenter size-medium wp-image-147" title="virt-manager" alt="" src="{{'assets/static/virt-manager.png' | prepend: site.baseurl | prepend: site.url }}" width="500" height="465" /></a>
