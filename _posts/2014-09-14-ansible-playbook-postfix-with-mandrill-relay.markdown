---
layout: post
status: publish
published: true
title: 'Ansible playbook: postfix with Mandrill relay'
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 264
wordpress_url: https://remote-lab.net/?p=264
date: '2014-09-14 12:52:15 +0000'
date_gmt: '2014-09-14 10:52:15 +0000'
categories:
- Linux
- Virtualization
tags: []
comments: []
---
<p>In this post I will show how you can use Ansible to automatically install postfix mail server and configure it to relay through Mandrill. Mandrill is a transactional email platform that allows you to send up to 12.000 emails for free. I use it for my servers to avoid situations where the IP addresses assigned by my ISP are blacklisted on some RBL lists. </p>
<p>Ansible is a config manamegement software that runs agentless over SSH. You only need python installed on the remote nodes. Ansible's configuration files are called playbooks. Playbooks are written as YAML files and they are used to manage configurations of and deployments to remote machines.</p>
<p>Below is the playbook that I use to install postfix, add the required configuration to use Mandrill and reload the service in order to use the new configuration. I will explain the playbook below:</p>
<p><code lang="bash[notools]"><br />
---<br />
- hosts: basenodes<br />
  tasks:<br />
    - name: Installs postfix mail server<br />
      apt: pkg=postfix state=installed update_cache=true<br />
      notify:<br />
        - start postfix<br />
    - name: Upload mandril authentication info<br />
      copy: src=/opt/files/postfix/mandril_passwd dest=/etc/postfix/mandril_passwd mode=0600<br />
      register: mandril<br />
      notify:<br />
        - postmap mandril_passwd<br />
    - name: Append mandril relay config<br />
      lineinfile:<br />
        dest=/etc/postfix/main.cf<br />
        line="{{ item.line }}"<br />
      with_items:<br />
        - { line: 'smtp_sasl_auth_enable = yes' }<br />
        - { line: 'smtp_sasl_password_maps = hash:/etc/postfix/mandril_passwd' }<br />
        - { line: 'smtp_sasl_security_options = noanonymous' }<br />
        - { line: 'smtp_use_tls = yes' }<br />
        - { line: 'relayhost = [smtp.mandrillapp.com]' }<br />
      notify:<br />
        - restart postfix</p>
<p>  handlers:<br />
    - name: start postfix<br />
      service: name=postfix state=started<br />
    - name: postmap mandril_passwd<br />
      command: postmap /etc/postfix/mandril_passwd<br />
      when: mandril|success<br />
    - name: restart postfix<br />
      service: name=postfix state=restarted<br />
</code></p>
<p>The hosts line contains the hosts group that this playbook will be applied to.<br />
Each play contains a list of tasks, which are actually calls to Ansible modules. We see that the first task is called 'Installs postfix mail server' and it uses the apt module to get the postfix package in the installed stated. update_cache=true ensures that 'apt-get update' will be run before installing the postfix package. The notify section contains the handlers. Handlers are lists of tasks, not really any different from regular tasks, that are referenced by name. You can find them in the handlers sections. Looking at our example - the 'start postfix' handler ensures that the postfix server is started. </p>
<p>The 'Upload mandril authentication info' task copies the /opt/files/postfix/mandril_passwd file on the ansible server to the remote node with /etc/postfix/mandril_passwd as a destination location. The mandril_passwd file contains the authentication details for the Mandril platform. The mode key contains the permissions the destination file will have. The register line gets the result of the copy operation stored in the mandril variable. After getting the file copied we need to create the postfix lookup table based on that file. In order to do this we run the 'postmap mandril_passwd' handler which runs the 'postmap /etc/postfix/mandril_passwd' command only if the copy task was run successfully.</p>
<p>The 'Append mandril relay config' task will add the config lines to the postfix main.cf files. We'll store the lines in a dictionary. A dictionary is represented in a simple key: and value form:. Each new line will be the value stored in the 'line' key of each dictionary element. After adding the lines to main.cf we'll restart postfix by running the 'restart postfix' handler. </p>
<p>You may find below the output of running the playbook : </p>
<p><code lang="bash[notools]"><br />
root@ansible:/etc/ansible/playbooks>>> ansible-playbook /etc/ansible/playbooks/playbook.yml</p>
<p>PLAY [basenodes] ************************************************************** </p>
<p>GATHERING FACTS *************************************************************** </p>
<p>TASK: [Installs postfix mail server] ******************************************<br />
changed: [node01.remote-lab.net]</p>
<p>TASK: [Upload mandril authentication info] ************************************<br />
changed: [node01.remote-lab.net]</p>
<p>TASK: [Append mandril relay config] *******************************************<br />
changed: [node01.remote-lab.net] => (item={'line': 'smtp_sasl_auth_enable = yes'})<br />
changed: [node01.remote-lab.net] => (item={'line': 'smtp_sasl_password_maps = hash:/etc/postfix/mandril_passwd'})<br />
changed: [node01.remote-lab.net] => (item={'line': 'smtp_sasl_security_options = noanonymous'})<br />
changed: [node01.remote-lab.net] => (item={'line': 'smtp_use_tls = yes'})<br />
changed: [node01.remote-lab.net] => (item={'line': 'relayhost = [smtp.mandrillapp.com]'})</p>
<p>NOTIFIED: [start postfix] *****************************************************<br />
ok: [node01.remote-lab.net]</p>
<p>NOTIFIED: [postmap mandril_passwd] ********************************************<br />
changed: [node01.remote-lab.net]</p>
<p>NOTIFIED: [restart postfix] ***************************************************<br />
changed: [node01.remote-lab.net]</p>
<p>PLAY RECAP ********************************************************************<br />
node01.remote-lab.net      : ok=7    changed=5    unreachable=0    failed=0<br />
</code></p>
