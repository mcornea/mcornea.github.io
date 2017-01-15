---
layout: post
comments: True
status: publish
published: true
title: LEMP stack HA cluster deployment with Ansible
date: '2016-03-26 10:21:31 +0000'
categories:
- Linux
- Openstack
- Ansible
permalink: ansible-lemp-ha-cluster
---

I've been playing with Ansible lately and I was thinking of a good practice exercise. The first thing that came to mind was an HA cluster since clustered nodes get almost identical configuration and most probably you want to use a config management system to deploy and don't touch them with any manual configuration. 

___

I chose to deploy a cluster that serves as the infrastructure for a LEMP stack (Linux, Nginx, MariaDB, PHP). 
This post will cover the setup details, a brief description of the Ansible roles that I used and some challenges that I faced.

<a href="{{'/public/images/lempha.jpg' | prepend: site.baseurl | prepend: site.url }}"><img src="{{'/public/images/lempha.jpg' | prepend: site.baseurl | prepend: site.url }}" alt="LEMP stack HA cluster" width="800" height="640"/></a>

## Cluster components:
 
- 3 x CentOS 7 nodes

Each node has 2 interfaces connected to one public network and one internal network. The public network is used for serving requests coming from the Internet. The internal one is used for communication such as database or file replication or communications between the loadbalancer and the backend servers.
   They are built as monolithic nodes, each one running all the services but it should be fairly easy to break them down based on roles.

- Keepalived: Virtual IP (VIP) address manager

In the eventuality of the node holding the VIP going down Keepalived brings up the VIP on another node in the cluster. We're going to make use of 2 x VIPs - one public VIP that is going to face the public network and will serve HTTP requests. A 2nd VIP on the internal network which serves the database requests.

- HAProxy: Loadbalancer

The Keepalived VIPs are used by HAProxy to expose the backends services. The public VIP has configured the 3 x Nginx servers as backends and provides roundrobin loadbalancing between them. The internal VIP has configured the 3 x MariaDB servers as backends. It is going to forward requests in an "active/passive" manner. This means that requests hitting the internal VIP will be routed to a single backend server. In the eventuality of the server going down then requests are routed to the next available backend server.

- MariaDB Galera cluster

The nodes form a multi-master synchronous MariaDB Galera cluster thus ensuring DB data consistency accross all nodes in the cluster.

- Nginx + php-fpm

We're going to use Nginx + php-fpm for running PHP applications. These can, of course, be interchanged with any other web server. The Nginx root directory is set to /var/www which is mounted from a GlusterFS volume.

- GlusterFS replicated volume

The gluster volume is set up as a 3 replicas volume backed up by 3 bricks, one brick per each node. 

## How it works:

Let's have a quick look at the flow that happens when you load a page in your browser:

  1. request hits the public VIP
  2. haproxy routes the request to one the nginx servers
  3. nginx uses php-fpm to run the php code stored in /var/www 
  4. the php script is loaded from one of the gluster bricks
  5. the code is evaluated and it runs a database query
  6. the query hits the internal vip 
  7. haproxy routes the query to the active backend database server
  7. the database query is run and the result is returned to the client

## Deploy!

Below are the Ansible roles that I used for provisioning:

 - base 
   
   simple role that installs common tools such as vim or git. In my case it also sets SELinux to permissive. Shame on me because I still haven't learnt SELinux :)
 - firewall 

   deploys and configures shorewall. It creates a firewall with 2 zones - one for the public network and one for the internal network. It also automatically allows all traffic between the nodes IP addresses and makes custom rules adjustable via group variables.
 - db 
   
    deploys and configures the Galera cluster
 - loadbalancer 
    
   deploys and configures HAProxy and Keepalived combo 
 - storage 

   creates the gluster bricks and volume and mounts it to /var/www
 - web 

   deploys and configures nginx + php-fpm 

I struggled a bit with the db role since I needed to serialize actions and I couldn't find a straightforward way of doing it. For example when a database server configuration change occurs you need to restart the mariadb service on each node individually and wait for it to get back in sync with the cluster. In order to do this I made a poor use of host variables. I'm pretty sure there must be a better way of achieving this and it's something that I'm looking to improve.

You can check the playbook <a href="https://github.com/remoteur/infra.remote-lab.net/tree/master/lemp-ha-cluster" target="_blank">here</a>

I'm quite impressed by Ansible's learning curve. I don't use it on a daily basis but every time I do I get the feeling that I can do complex tasks in a short timeframe which is awesome. 
