---
layout: post
comments: True
status: publish
published: true
title: Configuring users with admin role in OpenShift
date: '2019-01-18 10:21:31 +0000'
categories:
- Linux
- Docker
- OpenShift
- OKD
- Containers
permalink: openshift-cluster-admin-user
---

This is a quick post documenting the commands required for setting up Openshift users with the admin role:

{% highlight bash %}
## Admin within a project
oadm policy add-role-to-user admin $user -n $project
## Cluster wide admin
oadm policy add-cluster-role-to-user cluster-admin $user
{% endhighlight %}
