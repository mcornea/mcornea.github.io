---
layout: post
comments: True
status: publish
published: true
title: Running a Fedora CoreOS OpenStack instance
date: '2020-07-20 10:21:31 +0000'
categories:
- Linux
- Fedora CoreOS
- OpenShift
- Containers
permalink: fedora-coreos-openstack
---

Fedora CoreOS is a minimal operating system used for running containerized workloads that I've been wanting to experiment
with for some time. Since my test infra is OpenStack based I'll document below how to create a Fedora CoreOS OpenStack instance.

<center><a href="https://getfedora.org/en/coreos?stream=stable" target="_blank"><img src="https://fedoramagazine.org/wp-content/uploads/2019/07/introducing-fedora-coreos-816x345.png" alt="Fedora CoreOS" /></a></center>

___


*  Download the OpenStack Fedora CoreOS <a href="https://getfedora.org/en/coreos/download?tab=metal_virtualized&stream=stable" target="_blank">qcow image</a>


{% highlight bash %}
curl -O https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/32.20200629.3.0/x86_64/fedora-coreos-32.20200629.3.0-openstack.x86_64.qcow2.xz
{% endhighlight %}

*  Create Glance image

{% highlight bash %}
openstack image create \
  --disk-format qcow2 \
  --container-format bare \
  --public \
  --file fedora-coreos-32.20200629.3.0-openstack.x86_64.qcow2 \
  fedora-coreos-32.20200629.3.0
{% endhighlight %}

* Create an ignition config including any custom configuration that we want the instance to apply at boot time. In this example we pass the SSH key for the core user. For more advanced configurations check <a href="https://coreos.com/ignition/docs/latest/" target="_blank">Ignition documentation</a>

{% highlight bash %}
cat fedora_coreos.ign
{
  "ignition": {
    "version": "3.0.0"
  },
  "passwd": {
    "users": [
      {
        "name": "core",
        "sshAuthorizedKeys": [
          "ssh-rsa $YOUR__PUB_KEY“
        ]
      }
    ]
  }
}
{% endhighlight %}


*  Create instance
{% highlight bash %}
openstack server create 
  --image fedora-coreos-32.20200629.3.0  \
  --flavor m1.small \
  --security-group 49396992-488a-4ecf-acd5-58457aeff1ea \
  --network 6efea6c9-ad41-4a51-ac78-18e3654a779f \
  --user-data ./fedora_coreos.ign \
  --config-drive True \
  fedora_coreos
{% endhighlight %}

*  Assign a floating IP to the instance created in the step above
{% highlight bash %}
openstack server add floating ip fedora_coreos 172.16.32.103
{% endhighlight %}

*  Validate SSH connection to the instance by using the 'core' user
{% highlight bash %}
ssh core@172.16.32.103 'grep PRETTY_NAME /etc/os-release'
PRETTY_NAME="Fedora CoreOS 32.20200629.3.0"
{% endhighlight %}



