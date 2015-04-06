---
layout: post
title: Virtualized application infrastructure
date: '2014-05-07 12:32:41 +0000'
categories:
- Linux
- Virtualization
- Storage
permalink: virtualized-application-infrastructure
---

In this post I'll show how you can build a secure virtualized infrastructure for a basic webapp. We will break the setup into VMs that provide isolated services. You can find below the infrastructure diagram. The followings steps will show how you can set up a bare-metal server running Debian Wheezy to act as a KVM hypervisor and the process of deploying and configuring the VMs and the services they are running. 

___

<a href="{{'assets/static//rlug-New-Page.png' | prepend: site.baseurl | prepend: site.url }}"><img src="{{'assets/static//rlug-New-Page.png' | prepend: site.baseurl | prepend: site.url }}" alt="rlug - New Page" width="832" height="814" class="aligncenter size-full wp-image-255" /></a>

Install kvm and tools:
{% highlight bash %}
root@vmm:~>>> aptitude install qemu-kvm libvirt-bin virt-manager virt-viewer
{% endhighlight %} 

Install openvswitch :
{% highlight bash %}
root@vmm:~>>> aptitude install openvswitch-switch openvswitch-datapath-source
{% endhighlight %} 

Build Open vSwitch datapath kernel module:
{% highlight bash %}
root@vmm:~>>> module-assistant auto-install openvswitch-datapath
{% endhighlight %} 

The management IP address of the hypervisor and the other public IP addresses are assigned on the same interface by the hosting provider. In order to provide Internet connectivity for the VMs we need to create a bridge containing the physical interface where the public IPs are routed and add the VMs ports to this bridge. The trouble is that since this is also the management link we'll lose connectivity after adding the physical interface to the bridge. After this operation we need to assign the management IP address to the bridge interface. For doing this we'll edit the /etc/network/interfaces file.

Add openvswitch bridges:
{% highlight bash %}
root@vmm:~>>> ovs-vsctl add-br sw-net
{% endhighlight %} 

Edit the /etc/network/interfaces file:
{% highlight bash %}
root@vmm:~>>> cat /etc/network/interfaces
auto sw-net
iface sw-net inet static
  address   46.4.71.66
  broadcast 46.4.71.95
  netmask   255.255.255.224
  gateway   46.4.71.65
pre-up ip link set dev eth0 up
{% endhighlight %} 

At boot time the openvswitch daemon is started after the network init script so when the network init script is run it won't find the sw-net interface defined in /etc/network/interfaces file. A dirty workaround for this is to re-run the network init script after all the services are loaded. In order to do this we need to edit the /etc/rc.local file:  

{% highlight bash %}
root@vmm:~>>> cat /etc/rc.local
/etc/init.d/networking restart
exit 0
{% endhighlight %} 

Now let's add the physical interface to the bridge. After this we should either restart the network service from the console or do a hard reset:
{% highlight bash %}
root@vmm:~>>> ovs-vsctl add-port sw-net eth0
{% endhighlight %} 

The next step is to add the second bridge, where the internal network ports will be connected 
{% highlight bash %}
root@vmm:~>>> ovs-vsctl add-br sw-lan
{% endhighlight %} 

I prefer using virt-install for new VMs provisioning. The problem with it is that it currently doesn't support Open vSwitch bridges so we'll need to adjust it a little by adding the following line to the /usr/lib/pymodules/python2.7/virtinst/VirtualNetworkInterface.py file. This will add the virtualport tag to the VM xml definition:

{% highlight bash %}
root@vmm:~>>> diff -u /usr/lib/pymodules/python2.7/virtinst/VirtualNetworkInterface.py /usr/lib/pymodules/python2.7/virtinst/VirtualNetworkInterface.py.orig
--- /usr/lib/pymodules/python2.7/virtinst/VirtualNetworkInterface.py	2014-05-06 22:06:21.396072330 +0200
+++ /usr/lib/pymodules/python2.7/virtinst/VirtualNetworkInterface.py.orig	2014-05-06 22:13:17.121958858 +0200
@@ -384,7 +384,6 @@
         xml += "      <mac address='%s'/>\n" % self.macaddr
         xml += target_xml
         xml += model_xml
-        xml += "      <virtualport type='openvswitch'/>\n"
         xml += "    </interface>"
         return xml
{% endhighlight %} 

Now that we have the networking ready the last thing that we need are the storage files that the VMs will use. For creating the files we use the qemu-img utility. I prefer qcow2 files as they provide thin provision and snapshot capabilities. /var/lib/libvirt/images is the default directory used by libvirt so let's create the storage files here:

{% highlight bash %}
root@vmm:/var/lib/libvirt/images>>> qemu-img create -f qcow2 rtr01.qcow2 10G
Formatting 'rtr01.qcow2', fmt=qcow2 size=10737418240 encryption=off cluster_size=65536
{% endhighlight %} 

We can now create the VM and start the OS installation. We'll first install the rtr01 VM as it will provide Internet connectivity for the rest of the VMs in the internal network. The following command will generate a VM called rtr01 with 4 vCPUs, 4GB of ram, storage file located at /var/lib/libvirt/images/rtr01.qcow2 and 2 network interfaces - one in the bridge connected to the Internet and another connected to the internal network, the console is presented over VNC and it will first boot from the cdrom device loaded from the /var/lib/libvirt/images/vyatta-livecd_VC6.6R1_amd64.iso file. The disk and network interface will use paravirtualized drivers to obtain increased I/O performance.

{% highlight bash %}
root@vmm:~>>> virt-install --name rtr01 --vcpus=4 --ram=4096 --disk path=/var/lib/libvirt/images/rtr01.qcow2,bus=virtio --network bridge=sw-net,model=virtio --network bridge=sw-lan,model=virtio --graphics vnc --cdrom /var/lib/libvirt/images/vyatta-livecd_VC6.6R1_amd64.iso --boot cdrom
{% endhighlight %} 


After this command is issued a console windows will pop up and it will prompt the cdrom installation. After finishing the installation we can proceed to configuring the device:
{% highlight bash %}
# set interfaces IP addresses
set interfaces ethernet eth0 mac 00:50:56:00:5e:97
set interfaces ethernet eth0 address 46.4.71.77/27
set protocols static route 0.0.0.0/0 next-hop 46.4.71.65
set system host-name rtr01
set system domain-name nullzero.me
set interfaces ethernet eth1 10.0.1.1/24
set interfaces ethernet eth1 address 10.0.1.1/24
# set SNAT for internal network
set nat source rule 10 source address 10.0.1.0/24
set nat source rule 10 outbound-interface eth0
set nat source rule 10 translation address masquerade
# set DNAT for the request coming on port tcp 80 on the public IP
set nat destination rule 10 destination address 46.4.71.77
set nat destination rule 10 inbound-interface eth0
set nat destination rule 10 destination port 80
set nat destination rule 10 translation address 10.0.1.2
set nat destination rule 10 translation port 80
set nat destination rule 10 protocol tcp
# generate server and client certificates and keys
vyatta@rtr01:~$ sudo -s
vbash-4.1# cp -R /usr/share/doc/openvpn/examples/easy-rsa/2.0/* /etc/openvpn/
edit KEY_COUNTRY, KEY_PROVINCE, KEY_CITY, KEY_ORG, KEY_EMAIL variables
vbash-4.1# vi /etc/openvpn/vars
vbash-4.1# cd /etc/openvpn
vbash-4.1# source vars
vbash-4.1# ./clean-all
vbash-4.1# ./build-ca
vbash-4.1# ./build-dh
vbash-4.1# ./build-key-server rtr01
vbash-4.1# ./build-key client
vbash-4.1# mkdir /config/auth
vbash-4.1# cp -R /etc/openvpn/keys/* /config/auth
# configure the server certificates and key location
set interfaces openvpn vtun0 tls ca-cert-file /config/auth/ca.crt
set interfaces openvpn vtun0 tls cert-file /config/auth/rtr01.crt
set interfaces openvpn vtun0 tls dh-file /config/auth/dh1024.pem
set interfaces openvpn vtun0 tls key-file /config/auth/rtr01.key
# configure the openvpn server
set interfaces openvpn vtun0 mode server
set interfaces openvpn vtun0 server subnet 172.16.17.0/24
set interfaces openvpn vtun0 server push-route 10.0.1.0/24
set interfaces openvpn vtun0 openvpn-option "--comp-lzo --mssfix --tun-mtu 1488"
# openvpn client config file
marius@remoteur:~>>> cat /etc/openvpn/nullzero.conf
client
dev tun
proto udp
remote 46.4.71.77 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca /etc/openvpn/nullzero/ca.crt
cert /etc/openvpn/nullzero/client.crt
key /etc/openvpn/nullzero/client.key
ns-cert-type server
comp-lzo
verb 3
#configure firewall
set firewall state-policy established action 'accept'
set firewall state-policy related action 'accept'
set firewall all-ping 'enable'
edit firewall name rtr01
set default-action 'drop'
set rule 10 action accept
set rule 10 destination port 22
set rule 10 protocol tcp
set rule 11 action accept
set rule 11 destination port 80
set rule 11 protocol tcp
set rule 12 action accept
set rule 12 destination port 1194
set rule 12 protocol udp
exit
set interfaces ethernet eth0 firewall in name rtr01
{% endhighlight %} 

After completing these steps we should have a working router, firewall and VPN server.

Now let's continue with creating the second VM. We'll do a network install from minimal CD. First create the storage file:

{% highlight bash %}
root@vmm:~>>> qemu-img create -f qcow2 /var/lib/libvirt/images/lb01.qcow2 10G
Formatting '/var/lib/libvirt/images/lb01.qcow2', fmt=qcow2 size=10737418240 encryption=off cluster_size=65536
{% endhighlight %} 

Next we can start the installation process by using the cdrom file located at /var/lib/libvirt/images/debian-7.5.0-amd64-netinst.iso 

{% highlight bash %}
root@vmm:~>>> virt-install --name lb01 --vcpus=2 --ram=4096 --disk path=/var/lib/libvirt/images/lb01.qcow2,bus=virtio --network bridge=sw-lan,model=virtio  --graphics vnc --cdrom /var/lib/libvirt/images/debian-7.5.0-amd64-netinst.iso --boot cdrom
{% endhighlight %} 

After completing the OS installation we have a fresh running Debian Wheezy system. We don't want to repeat the install process for the other files so we'll just copy the existing image of the Debian system and modify the IP settings and hostnames. We first copy the base image, then attach it by using qemu-nbd, mount the partition where the file system resides and then edit the files that we need.

{% highlight bash %}
root@vmm:/var/lib/libvirt/images>>> cp lb01.qcow2 db01.qcow2; cp lb01.qcow2 web01.qcow2
root@vmm:/var/lib/libvirt/images>>> modprobe nbd max_part=8
root@vmm:/var/lib/libvirt/images>>> qemu-nbd -c /dev/nbd0 web01.qcow2
root@vmm:/var/lib/libvirt/images>>> kpartx -a /dev/nbd0
root@vmm:/var/lib/libvirt/images>>> mount /dev/mapper/nbd0p1 /mnt
root@vmm:/var/lib/libvirt/images>>> vim /mnt/etc/network/interfaces
root@vmm:/var/lib/libvirt/images>>> vim /mnt/etc/hosts
root@vmm:/var/lib/libvirt/images>>> vim /mnt/etc/hostname
root@vmm:/var/lib/libvirt/images>>> umount /mnt
root@vmm:/var/lib/libvirt/images>>> kpartx -d /dev/nbd0
root@vmm:/var/lib/libvirt/images>>> qemu-nbd -d /dev/nbd0
{% endhighlight %} 

We repeat the steps above for the db01.qcow2 file.
Let's now create the web01 and db01 VMs. Since we already have the base storage files we don't need to run the OS installation:

{% highlight bash %}
root@vmm:~>>> virt-install --name web01 --vcpus=4 --ram=4096 --disk path=/var/lib/libvirt/images/web01.qcow2,bus=virtio --network bridge=sw-lan,model=virtio  --graphics vnc --import
root@vmm:~>>> virt-install --name db01 --vcpus=4 --ram=4096 --disk path=/var/lib/libvirt/images/db01.qcow2,bus=virtio --network bridge=sw-lan,model=virtio  --graphics vnc --import
{% endhighlight %} 

Once we have booted al the VMs let's start configuring the services.
On the http load balancer we'll install varnish and configure the web server as backend: 

{% highlight bash %}
root@lb01:~>>> aptitude install varnish
root@lb01:~>>> sed -i 's/6081/80/' /etc/default/varnish
root@lb01:~>>> sed -i 's/127.0.0.1/10.0.1.3/' /etc/varnish/default.vcl
root@lb01:~>>> sed -i 's/8080/80/' /etc/varnish/default.vcl
root@lb01:~>>> /etc/init.d/varnish restart
{% endhighlight %} 

On the web server we'll install nginx and php-fpm and configure the default vhost:
{% highlight bash %}
root@web01:~>>> aptitude install nginx php5-fpm php5-mysql
{% endhighlight %}

Add the following location block to the first server block:
{% highlight bash %}
    location ~ \.php$ {
        fastcgi_pass   unix:/var/run/php5-fpm.sock;
        fastcgi_index  index.php;
        include        fastcgi_params;
}
{% endhighlight %} 

Create an index file in the document root that will query the database server:

{% highlight bash %}
root@web01:/srv/www>>> cat index.php
<?php
$con=mysqli_connect("10.0.1.4","user","parola","test");
$result = mysqli_query($con,"SELECT * FROM testable");
$row = mysqli_fetch_array($result);
echo $row['hello'];
mysqli_close($con);
?>
{% endhighlight %} 

On the database server we'll install mysql server and create a dummy database and table;

{% highlight bash %}
root@db01:~>>> aptitude install mysql-server
root@db01:~>>> mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 43
Server version: 5.5.37-0+wheezy1 (Debian)
Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> create database test;
mysql> use test;
mysql> CREATE TABLE testable (hello VARCHAR(20));
mysql> INSERT INTO testable (hello) VALUES("Hello World!");
mysql> CREATE USER 'user'@'10.0.1.3' IDENTIFIED BY 'parola';
mysql> GRANT ALL PRIVILEGES ON * . * TO 'user'@'10.0.1.3';
mysql> FLUSH PRIVILEGES;
mysql> exit
{% endhighlight %} 

After this final step we have our setup ready and http://app.nullzero.me/ should show the Hello World!
