---
layout: post
status: publish
published: true
title: OSPF lab provisioning on IOS with Ansible
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 296
wordpress_url: https://remote-lab.net/?p=296
date: '2015-02-24 19:21:31 +0000'
date_gmt: '2015-02-24 17:21:31 +0000'
categories:
- Uncategorized
- Linux
- Routing
- IOS
- Python
tags: []
comments:
- id: 17631
  author: Paul Chandler
  author_email: Paul@Chandlerweb.co.uk
  author_url: ''
  date: '2015-02-26 17:38:27 +0000'
  date_gmt: '2015-02-26 15:38:27 +0000'
  content: Great stuff! Hopefully I'll get time to go through this and try it out
    myself.
---
<p>In this post we'll see how we can quickly get a basic OSPF lab deployed by using Ansible. Our setup consists of 3 x Cisco IOS routers which are connected according to the diagram below. All the routers should already have SSH set up and an interface connected to the management network that will be used for retrieving the configuration files from the server. On the server side we need a Linux machine that has Ansible installed.</p>
<p><a href="https://remote-lab.net/wp-content/uploads/2015/02/ansible_ospf-New-Page.png"><img src="https://remote-lab.net/wp-content/uploads/2015/02/ansible_ospf-New-Page.png" alt="ansible_ospf - New Page" width="649" height="594" class="aligncenter size-full wp-image-297" /></a></p>
<p>So let's get started by building the Ansible playbook. I'll explain the site.yml file below: </p>
<p>1. Run the playbook on the 'localhost' server by using the 'marius' username with sudo<br />
2. Install and start Apache as the routers will pull the config files over HTTP<br />
3. Install Git - used to clone the netmiko library repo<br />
4. Install paramiko - dependency library for netmiko<br />
5. Clone the netmiko repo and install it as a system module<br />
6. Next we use the netmiko.j2 template to create a script file. The script uses netmiko and takes as arguments the ip address, username, password and command that will be run on the remote Cisco device.<br />
7. We generate the configuration files that are going to be pulled by the routers. For this we use the config.j2 template and write the configuration files on the Apache DocumentRoot. This will results in 3 files: /var/www/html/rtr-A.config, /var/www/html/rtr-B.config, /var/www/html/rtr-C.config containing the configuration commands for each of the routers.<br />
8. We run the script that we have created on step 6 by passing the 'copy http://... running-config' command to each of the routers.<br />
9. Define the variables used in the template files and commands.</p>
<p><code lang="bash[notools]"><br />
---<br />
- hosts: localhost<br />
  remote_user: marius<br />
  sudo: yes</p>
<p>  tasks:<br />
  - name: Install Apache<br />
    yum: name=httpd state=installed</p>
<p>  - name: Start Apache<br />
    service: name=httpd state=started enabled=yes</p>
<p>  - name: Install Git<br />
    yum: name=git state=installed</p>
<p>  - name: Install paramiko<br />
    yum: name=python-paramiko state=installed</p>
<p>  - name: Clone netmiko<br />
    git: repo=https://github.com/ktbyers/netmiko.git dest=/opt/netmiko</p>
<p>  - name: Install netmiko<br />
    command: chdir=/opt/netmiko python setup.py install</p>
<p>  - name: Create netmiko script<br />
    template: src=netmiko.j2 dest=/opt/remote_cmd</p>
<p>  - name: Generate config file<br />
    template: src=config.j2 dest=/var/www/html/{{ item.name }}.config<br />
    with_items: routers</p>
<p>  - name: Connect to routers and pull the config<br />
    shell: python /opt/remote_cmd {{ item.mgmt_ip }} {{ item.mgmt_user }} {{ item.mgmt_pass }} 'copy http://{{ ansible_eth0.ipv4.address }}/{{ item.name }}.config running-config'<br />
    with_items: routers</p>
<p>  vars:<br />
   routers:<br />
     - name: "rtr-A"<br />
       mgmt_ip: "192.168.0.81"<br />
       mgmt_user: "admin"<br />
       mgmt_pass: "parola"<br />
       int:<br />
         - name: "GigabitEthernet0/1"<br />
           address: "10.0.0.1"<br />
           netmask: "255.255.255.252"<br />
           ospf: "yes"<br />
         - name: "GigabitEthernet0/2"<br />
           address: "10.0.0.5"<br />
           netmask: "255.255.255.252"<br />
           ospf: "yes"<br />
         - name: "Loopback0"<br />
           address: "1.1.1.1"<br />
           netmask: "255.255.255.0"<br />
           ospf: "yes"</p>
<p>     - name: "rtr-B"<br />
       mgmt_ip: "192.168.0.78"<br />
       mgmt_user: "admin"<br />
       mgmt_pass: "parola"<br />
       int:<br />
        - name: "GigabitEthernet0/1"<br />
          address: "10.0.0.2"<br />
          netmask: "255.255.255.252"<br />
          ospf: "yes"<br />
        - name: "GigabitEthernet0/2"<br />
          address: "10.0.0.9"<br />
          netmask: "255.255.255.252"<br />
          ospf: "yes"<br />
        - name: "Loopback0"<br />
          address: "2.2.2.2"<br />
          netmask: "255.255.255.0"<br />
          ospf: "yes"</p>
<p>     - name: "rtr-C"<br />
       mgmt_ip: "192.168.0.79"<br />
       mgmt_user: "admin"<br />
       mgmt_pass: "parola"<br />
       int:<br />
         - name: "GigabitEthernet0/1"<br />
           address: "10.0.0.6"<br />
           netmask: "255.255.255.252"<br />
           ospf: "yes"<br />
         - name: "GigabitEthernet0/2"<br />
           address: "10.0.0.10"<br />
           netmask: "255.255.255.252"<br />
           ospf: "yes"<br />
         - name: "Loopback0"<br />
           address: "3.3.3.3"<br />
           netmask: "255.255.255.0"<br />
           ospf: "no" </code></p>
<p>Now let's go through the template files. </p>
<p>The config.j2 template is used to build the configuration commands that will be loaded by the routers. What this does is basically loop through the interfaces defined for each of the routers and create the ip address statements for each of them. After this, it generates an entry for the ospf process and creates a network statement if the 'ospf' variable is set to yes for a specific interface. </p>
<p><code lang="bash[notools]"><br />
config.j2<br />
{% for iface in item.int %}<br />
interface {{ iface.name }}<br />
no shutdown<br />
ip address {{ iface.address }} {{ iface.netmask }}<br />
{% endfor %}</p>
<p>router ospf 1<br />
{% for iface in item.int %}<br />
{% if iface.ospf == "yes" %}<br />
network {{ iface.address }} 0.0.0.0 area 0<br />
{% endif %}<br />
{% endfor %}</code></p>
<p>The netmiko.j2 template is just a python script that's using netmiko to connect to the router, first runs 'file prompt quiet' configuration command to disable the save confirmation message. Then it runs the command that's passed as the 4th argument.  </p>
<p><code lang="bash[notools]"><br />
netmiko.j2<br />
#!/usr/bin/env python<br />
import netmiko<br />
import sys</p>
<p>cisco_881 = {<br />
	'device_type': 'cisco_ios',<br />
	'ip':   sys.argv[1],<br />
	'username': sys.argv[2],<br />
	'password': sys.argv[3],<br />
	'secret': 'secret',<br />
	'verbose': False,<br />
}</p>
<p>SSHClass = netmiko.ssh_dispatcher(device_type=cisco_881['device_type'])</p>
<p>net_connect = SSHClass(**cisco_881)</p>
<p>config_commands = [ 'file prompt quiet' ]<br />
output = net_connect.send_config_set(config_commands)</p>
<p>output = net_connect.send_command(sys.argv[4])<br />
</code></p>
<p>In order to run this we will also need the /etc/ansible/hosts file to contain the localhost hostname. You can find all the files in this GitHub repo: <a href="https://github.com/remoteur/ansible-ospflab.git">https://github.com/remoteur/ansible-ospflab.git</a></p>
<p>Once we have all the files in place we can run the playbook by the 'ansible-playbook site.yml' command. This is how the output looks like:</p>
<p><code lang="bash[notools]"><br />
marius@net-master:/etc/ansible>>> ansible-playbook site.yml </p>
<p>PLAY [localhost] ************************************************************** </p>
<p>GATHERING FACTS ***************************************************************<br />
ok: [localhost]</p>
<p>TASK: [Install Apache] ********************************************************<br />
ok: [localhost]</p>
<p>TASK: [Start Apache] **********************************************************<br />
ok: [localhost]</p>
<p>TASK: [Install Git] ***********************************************************<br />
ok: [localhost]</p>
<p>TASK: [Install paramiko] ******************************************************<br />
ok: [localhost]</p>
<p>TASK: [Clone netmiko] *********************************************************<br />
ok: [localhost]</p>
<p>TASK: [Install netmiko] *******************************************************<br />
changed: [localhost]</p>
<p>TASK: [Create netmiko script] *************************************************<br />
ok: [localhost]</p>
<p>TASK: [Generate config file] **************************************************<br />
ok: [localhost] => (item={'mgmt_pass': 'parola', 'int': [{'netmask': '255.255.255.252', 'ospf': 'yes', 'name': 'GigabitEthernet0/1', 'address': '10.0.0.1'}, {'netmask': '255.255.255.252', 'ospf': 'yes', 'name': 'GigabitEthernet0/2', 'address': '10.0.0.5'}, {'netmask': '255.255.255.0', 'ospf': 'yes', 'name': 'Loopback0', 'address': '1.1.1.1'}], 'name': 'rtr-A', 'mgmt_ip': '192.168.0.81', 'mgmt_user': 'admin'})<br />
ok: [localhost] => (item={'mgmt_pass': 'parola', 'int': [{'netmask': '255.255.255.252', 'ospf': 'yes', 'name': 'GigabitEthernet0/1', 'address': '10.0.0.2'}, {'netmask': '255.255.255.252', 'ospf': 'yes', 'name': 'GigabitEthernet0/2', 'address': '10.0.0.9'}, {'netmask': '255.255.255.0', 'ospf': 'yes', 'name': 'Loopback0', 'address': '2.2.2.2'}], 'name': 'rtr-B', 'mgmt_ip': '192.168.0.78', 'mgmt_user': 'admin'})<br />
ok: [localhost] => (item={'mgmt_pass': 'parola', 'int': [{'netmask': '255.255.255.252', 'ospf': 'yes', 'name': 'GigabitEthernet0/1', 'address': '10.0.0.6'}, {'netmask': '255.255.255.252', 'ospf': 'yes', 'name': 'GigabitEthernet0/2', 'address': '10.0.0.10'}, {'netmask': '255.255.255.0', 'ospf': 'no', 'name': 'Loopback0', 'address': '3.3.3.3'}], 'name': 'rtr-C', 'mgmt_ip': '192.168.0.79', 'mgmt_user': 'admin'})</p>
<p>TASK: [Connect to routers and pull the config] ********************************<br />
changed: [localhost] => (item={'mgmt_pass': 'parola', 'int': [{'netmask': '255.255.255.252', 'ospf': 'yes', 'name': 'GigabitEthernet0/1', 'address': '10.0.0.1'}, {'netmask': '255.255.255.252', 'ospf': 'yes', 'name': 'GigabitEthernet0/2', 'address': '10.0.0.5'}, {'netmask': '255.255.255.0', 'ospf': 'yes', 'name': 'Loopback0', 'address': '1.1.1.1'}], 'name': 'rtr-A', 'mgmt_ip': '192.168.0.81', 'mgmt_user': 'admin'})<br />
changed: [localhost] => (item={'mgmt_pass': 'parola', 'int': [{'netmask': '255.255.255.252', 'ospf': 'yes', 'name': 'GigabitEthernet0/1', 'address': '10.0.0.2'}, {'netmask': '255.255.255.252', 'ospf': 'yes', 'name': 'GigabitEthernet0/2', 'address': '10.0.0.9'}, {'netmask': '255.255.255.0', 'ospf': 'yes', 'name': 'Loopback0', 'address': '2.2.2.2'}], 'name': 'rtr-B', 'mgmt_ip': '192.168.0.78', 'mgmt_user': 'admin'})<br />
changed: [localhost] => (item={'mgmt_pass': 'parola', 'int': [{'netmask': '255.255.255.252', 'ospf': 'yes', 'name': 'GigabitEthernet0/1', 'address': '10.0.0.6'}, {'netmask': '255.255.255.252', 'ospf': 'yes', 'name': 'GigabitEthernet0/2', 'address': '10.0.0.10'}, {'netmask': '255.255.255.0', 'ospf': 'no', 'name': 'Loopback0', 'address': '3.3.3.3'}], 'name': 'rtr-C', 'mgmt_ip': '192.168.0.79', 'mgmt_user': 'admin'})</p>
<p>PLAY RECAP ********************************************************************<br />
localhost                  : ok=10   changed=2    unreachable=0    failed=0<br />
</code></p>
<p>This is how one of routers configuration files looks like:</p>
<p><code lang="bash[notools]"><br />
marius@net-master:/etc/ansible>>> cat /var/www/html/rtr-A.config<br />
interface GigabitEthernet0/1<br />
no shutdown<br />
ip address 10.0.0.1 255.255.255.252<br />
interface GigabitEthernet0/2<br />
no shutdown<br />
ip address 10.0.0.5 255.255.255.252<br />
interface Loopback0<br />
no shutdown<br />
ip address 1.1.1.1 255.255.255.0</p>
<p>router ospf 1<br />
network 10.0.0.1 0.0.0.0 area 0<br />
network 10.0.0.5 0.0.0.0 area 0<br />
network 1.1.1.1 0.0.0.0 area 0<br />
</code></p>
<p>So now we have the lab up and running. Why bother automating this? In the end it's a basic test environment. Here's my motivation:</p>
<p>1. I hate doing repetitive stuff<br />
2. Reproducibility. Manual repetitive stuff results in errored configurations, at least for me. If I do such a setup manually I usually get a terminal started for each of the routers and start writing commands. My problem is that almost all the time I end up messing up something like setting the wrong IP addresses on interfaces. By running this playbook I will always get the same result.<br />
3. I have the complete picture in one place and I can check the whole setup before running it, no need to switch through terminals, screens or other stuff.<br />
4. Time. I'm running this setup on Openstack by using Cisco vIOS images so getting everything up and running from scratch takes me less than 5 minutes which is pretty awesome.</p>
<p>Let me know if you have any questions and I'll be more than happy to answer. </p>
<p>Thanks</p>
