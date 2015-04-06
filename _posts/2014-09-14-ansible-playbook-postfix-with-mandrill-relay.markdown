---
layout: post
title: 'Ansible playbook: postfix with Mandrill relay'
date: '2014-09-14 12:52:15 +0000'
categories:
- Linux
- Virtualization
- Ansible
permalink: ansible-postfix-with-mandrill
---

In this post I will show how you can use Ansible to automatically install postfix mail server and configure it to relay through Mandrill. Mandrill is a transactional email platform that allows you to send up to 12.000 emails for free. I use it for my servers to avoid situations where the IP addresses assigned by my ISP are blacklisted on some RBL lists. 

___

Ansible is a config manamegement software that runs agentless over SSH. You only need python installed on the remote nodes. Ansible's configuration files are called playbooks. Playbooks are written as YAML files and they are used to manage configurations of and deployments to remote machines.

Below is the playbook that I use to install postfix, add the required configuration to use Mandrill and reload the service in order to use the new configuration. I will explain the playbook below:

{% gist aa3d27a6fc948ed6e857 %}

The hosts line contains the hosts group that this playbook will be applied to.
Each play contains a list of tasks, which are actually calls to Ansible modules. We see that the first task is called 'Installs postfix mail server' and it uses the apt module to get the postfix package in the installed stated. update_cache=true ensures that 'apt-get update' will be run before installing the postfix package. The notify section contains the handlers. Handlers are lists of tasks, not really any different from regular tasks, that are referenced by name. You can find them in the handlers sections. Looking at our example - the 'start postfix' handler ensures that the postfix server is started. 

The 'Upload mandril authentication info' task copies the /opt/files/postfix/mandril_passwd file on the ansible server to the remote node with /etc/postfix/mandril_passwd as a destination. The mandril_passwd file contains the authentication details for the Mandril platform. The mode key contains the permissions the destination file will have. The register line gets the result of the copy operation stored in the mandril variable. After getting the file copied we need to create the postfix lookup table based on that file. In order to do this we run the 'postmap mandril_passwd' handler which runs the 'postmap /etc/postfix/mandril_passwd' command only if the copy task was run successfully.

The 'Append mandril relay config' task will add the config lines to the postfix main.cf files. We'll store the lines in a dictionary. A dictionary is represented in a simple key: and value form:. Each new line will be the value stored in the 'line' key of each dictionary element. After adding the lines to main.cf we'll restart postfix by running the 'restart postfix' handler. 

You may find below the output of running the playbook : 


{% highlight bash %}
root@ansible:/etc/ansible/playbooks>>> ansible-playbook /etc/ansible/playbooks/playbook.yml
PLAY [basenodes] ************************************************************** 
GATHERING FACTS *************************************************************** 
TASK: [Installs postfix mail server] ******************************************
changed: [node01.remote-lab.net]
TASK: [Upload mandril authentication info] ************************************
changed: [node01.remote-lab.net]
TASK: [Append mandril relay config] *******************************************
changed: [node01.remote-lab.net] => (item={'line': 'smtp_sasl_auth_enable = yes'})
changed: [node01.remote-lab.net] => (item={'line': 'smtp_sasl_password_maps = hash:/etc/postfix/mandril_passwd'})
changed: [node01.remote-lab.net] => (item={'line': 'smtp_sasl_security_options = noanonymous'})
changed: [node01.remote-lab.net] => (item={'line': 'smtp_use_tls = yes'})
changed: [node01.remote-lab.net] => (item={'line': 'relayhost = [smtp.mandrillapp.com]'})
NOTIFIED: [start postfix] *****************************************************
ok: [node01.remote-lab.net]
NOTIFIED: [postmap mandril_passwd] ********************************************
changed: [node01.remote-lab.net]
NOTIFIED: [restart postfix] ***************************************************
changed: [node01.remote-lab.net]
PLAY RECAP ********************************************************************
node01.remote-lab.net      : ok=7    changed=5    unreachable=0    failed=0
{% endhighlight %}
