---
layout: post
comments: True
status: publish
published: true
title: Deploying all-in-one OKD 3.11 with Let's Encrypt SSL certificates
date: '2019-01-15 10:21:31 +0000'
categories:
- Linux
- Docker
- OpenShift
- OKD
- Containers
permalink: aio-okd-311-lets-encrypt
---

I've been experimenting with OpenShift lately and in the following post I'd like to document the steps required to get an OKD 3.11 environment up and running. For those who are not familiar with the OKD abbreviation it is "The Origin Community Distribution of Kubernetes that powers Red Hat OpenShift". The deployment is going to be set up on a single node and configured to use Let's Encrypt SSL certificates for the API endpoint/console and HTTPS routes.

___


The main purpose of this exercise is to:

  * get myself familiar with the openshift-ansible installer process
  * get an OpenShift environment connected to the Internet up and running that I can experiment with

Let's get started.

### Prerequisites

#### Hardware
** Get a beefy CentOS 7 VM ready. For this test I used one with 4 vCPUs, 16 GB of RAM and 50GB of storage. The specs could be lowered but you may need to disable specific <a href="https://github.com/openshift/openshift-ansible/blob/master/inventory/hosts.localhost#L14" target="_blank">checks</a> the installer runs.

#### DNS configuration
** I’m going to use ‘containers.remote-lab.net’ as the domain name for this OpenShift installation. Consequently I’ve got to set up the following DNS records in my domain’s zone file to be able to reach the environment from any client. The wildcard mask entry is required for the OpenShift routes:

{% highlight bash %}
containers.remote-lab.net.	   IN	A	148.251.139.92
*.containers.remote-lab.net.       IN	A	148.251.139.92
{% endhighlight %}

___

### Prepare

* Set up the hostname

{% highlight bash %}
## SSH to the CentOS VM
[centos@containers ~]$ sudo hostnamectl set-hostname containers.remote-lab.net
[centos@containers ~]$ sudo hostnamectl set-hostname containers.remote-lab.net --transient
[centos@containers ~]$ hosts=$(echo "$(ip a s dev eth0 | awk '/inet / {split($2, ary, /\//); print ary[1]}') $(hostname -f) $(hostname -s)")
[centos@containers ~]$ sudo sh -c "echo $hosts >> /etc/hosts"
{% endhighlight %}

* Set up the OKD 3.11 repos

{% highlight bash %}
[centos@containers ~]$ sudo sh -c 'cat > /etc/yum.repos.d/openshift.repo << EOF
[openshift-3.11]
name=OpenShift Origin 3.11
baseurl=https://cbs.centos.org/repos/paas7-openshift-origin311-candidate/\$basearch//os/
enabled=1
gpgcheck=0

[openshift-common]
name=OpenShift Common
baseurl=https://cbs.centos.org/repos/paas7-openshift-common-candidate/\$basearch//os/
enabled=1
gpgcheck=0
EOF'
{% endhighlight %}


* Install openshift-ansible and enable NetworkManager
{% highlight bash %}
[centos@containers ~]$ sudo yum install -y openshift-ansible NetworkManager pyOpenSSL
[centos@containers ~]$ sudo systemctl start NetworkManager
[centos@containers ~]$ sudo systemctl enable NetworkManager
{% endhighlight %}

* Fix small dependency issue if still necessary

{% highlight bash %}
[centos@containers ~]$ curl -L https://github.com/openshift/openshift-ansible/pull/10812/commits/0d7608fedc522a8d034191c4e07d880b82e07006.diff | sudo patch -d /usr/share/ansible/openshift-ansible/ -p1
{% endhighlight %}


* Generate Let’s Encrypt certificates. I'm using Cloudflare for hosting my domain DNS zone so the example below will call the Cloudflare script

{% highlight bash %}
[centos@containers ~]$ sudo yum install -y git
[centos@containers ~]$ git clone https://github.com/Neilpang/acme.sh.git
[centos@containers ~]$ cd acme.sh/
## set your API key and email address in dnsapi/dns_cf.sh
[centos@containers acme.sh]$ vi dnsapi/dns_cf.sh
[centos@containers acme.sh]$ ./acme.sh --issue -d containers.remote-lab.net -d *.containers.remote-lab.net --dns dns_cf
## copy certificate and key that got created in the previous step
[centos@containers acme.sh]$ sudo cp /home/centos/.acme.sh/containers.remote-lab.net/containers.remote-lab.net.cer /etc/pki/tls/certs/containers.remote-lab.net.cer
[centos@containers acme.sh]$ sudo cp /home/centos/.acme.sh/containers.remote-lab.net/containers.remote-lab.net.key /etc/pki/tls/private/containers.remote-lab.net.key
[centos@containers acme.sh]$ sudo cp /home/centos/.acme.sh/containers.remote-lab.net/ca.cer /etc/pki/tls/certs/containers.remote-lab.net.ca.cer
{% endhighlight %}

* Create Inventory file
{% highlight bash %}
[centos@containers ~]$ cat okd-inventory

[OSEv3:children]
masters
nodes
etcd

[OSEv3:vars]
ansible_connection=local
openshift_deployment_type=origin

openshift_master_identity_providers=[{ 'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]
openshift_master_htpasswd_users={'marius': '$apr1$fQGIonSg$Y6vvC8Q4Lowzp0EQl.mlE1'}

openshift_master_cluster_hostname=containers.remote-lab.net
openshift_master_cluster_public_hostname=containers.remote-lab.net
openshift_master_default_subdomain=containers.remote-lab.net

openshift_master_overwrite_named_certificates=true
openshift_master_named_certificates=[{"certfile": "/etc/pki/tls/certs/containers.remote-lab.net.cer", "keyfile": "/etc/pki/tls/private/containers.remote-lab.net.key", "names": ["containers.remote-lab.net"], "cafile": "/etc/pki/tls/certs/containers.remote-lab.net.ca.cer"}]

openshift_cluster_monitoring_operator_install=false
openshift_metrics_install_metrics=false
ansible_service_broker_install=false

[masters]
containers.remote-lab.net

[etcd]
containers.remote-lab.net

[nodes]
containers.remote-lab.net openshift_node_group_name='node-config-all-in-one'

[oo_all_hosts:children]
masters
nodes
etcd
{% endhighlight %}

___

### Deploy!

* Run prerequisites playbook

{% highlight bash %}
[centos@containers ~]$ sudo ansible-playbook -i okd-inventory /usr/share/ansible/openshift-ansible/playbooks/prerequisites.yml
{% endhighlight %}

* Run deploy playbook
{% highlight bash %}
[centos@containers ~]$ sudo ansible-playbook -i okd-inventory /usr/share/ansible/openshift-ansible/playbooks/deploy_cluster.yml
{% endhighlight %}

* After running the deploy_cluster playbook the OpenShift setup should be up and running and reachable from the Internet.

{% highlight bash %}
[centos@containers ~]$ sudo oc status
In project default on server https://containers.remote-lab.net:8443

https://docker-registry-default.containers.remote-lab.net (passthrough) (svc/docker-registry)
  dc/docker-registry deploys docker.io/openshift/origin-docker-registry:v3.11.0 
    deployment #2 deployed 14 minutes ago - 1 pod
    deployment #1 deployed 26 minutes ago

svc/kubernetes - 172.30.0.1 ports 443->8443, 53->8053, 53->8053

https://registry-console-default.containers.remote-lab.net (passthrough) (svc/registry-console)
  dc/registry-console deploys docker.io/cockpit/kubernetes:latest 
    deployment #1 deployed 26 minutes ago - 1 pod

svc/router - 172.30.42.93 ports 80, 443, 1936
  dc/router deploys docker.io/openshift/origin-haproxy-router:v3.11.0 
    deployment #2 deployed 15 minutes ago - 1 pod
    deployment #1 deployed 26 minutes ago

View details with 'oc describe <resource>/<name>' or list everything with 'oc get all'.
{% endhighlight %}

* There is one more step that we have to do post deployment. This replaces the current router certficate with the Let's Encrypt certificate that we created earlier.

{% highlight bash %}
[centos@containers ~]$ cat /etc/pki/tls/certs/containers.remote-lab.net.cer /etc/pki/tls/private/containers.remote-lab.net.key /etc/pki/tls/certs/containers.remote-lab.net.ca.cer > containers.remote-lab.net.pem
[centos@containers ~]$ sudo sh -c "oc secrets new router-certs tls.crt=/home/centos/containers.remote-lab.net.pem tls.key=/etc/pki/tls/private/containers.remote-lab.net.key -o json --type='kubernetes.io/tls' --confirm | oc replace -f -"
{% endhighlight %}

___

### Enjoy!

The OpenShift console can now be reached @ <a href="https://containers.remote-lab.net:8443" target="_blank">https://containers.remote-lab.net:8443</a>.

Credit goes to Marcos Entenza for sharing the Let's Encrypt certificate <a href="http://maklog.io/post/free-wildcard-certs-openshift/" target="_blank">instructions</a>.
