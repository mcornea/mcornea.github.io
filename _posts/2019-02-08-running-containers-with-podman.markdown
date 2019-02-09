---
layout: post
comments: True
status: publish
published: true
title: Running containers with Podman
date: '2019-02-8 10:21:31 +0000'
categories:
- Linux
- OpenShift
- Containers
permalink: podman-containers
---

In this post I'll document how to build and run containers with Podman. Podman 
is a tool which allows you managing OCI containers without the need for a daemon. 
It aims to provide parity with the docker cli options making it easy for users 
to transition to using this new tool. You can read more about the Podman motivation
in this detailed <a href="https://www.projectatomic.io/blog/2018/02/reintroduction-podman/" target="_blank">article</a>.

<center><a href="https://podman.io/" target="_blank"><img src="{{'/public/images/podman.png' | prepend: site.baseurl | prepend: site.url }}" alt="Podman" /></a></center>

___

I ran the steps below on a Fedora 29 system and used an nginx container for this
exercise.


{% highlight bash %}
## Install podman package
sudo dnf install -y podman

## Create a dockerfile and example conf

cat >> Dockerfile << EOF
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
EOF

cat >> nginx.conf << EOF
 events {
     worker_connections  1024;
 }

 http {


 upstream test {
   server 192.168.0.21:80;
 }

 server {
   listen 80;
   server_name example.com;
   location / {
     proxy_pass http://test;
   }
 }

 }
EOF

## Build image
podman build -t nginx .

## List local images
podman images
REPOSITORY                TAG      IMAGE ID       CREATED          SIZE
localhost/nginx           latest   15b6754f2c6a   20 seconds ago   113 MB
<none>                    <none>   f0d066b2080f   21 seconds ago   113 MB
docker.io/library/nginx   latest   f09fe80eb0e7   2 days ago       113 MB

## Run container
podman run -d --name nginx -p 80:80 -t localhost/nginx
6744d22d25e7dd93b72e3f907d9174bdac573ee423a865187fb82dee3db3c0f5

## List running containers
podman ps
CONTAINER ID  IMAGE                   COMMAND               CREATED        STATUS            PORTS  NAMES
6744d22d25e7  localhost/nginx:latest  nginx -g daemon o...  3 seconds ago  Up 3 seconds ago         nginx

{% endhighlight %}

As you can notice the user experience remains pretty much the same as with the 
docker cli. One of the differences in terms of operational experience is how the 
containers start at boot time. Since there's no daemon involved in managing the 
containers we have to rely on systemd and create unit files for each container 
to automatically start at boot:

{% highlight bash %}
## Create systemd unit file
cat >> /etc/systemd/system/nginx.service << EOF
[Unit]
Description=Nginx Podman container
Wants=syslog.service
[Service]
Restart=always
ExecStart=/usr/bin/podman start -a nginx
ExecStop=/usr/bin/podman stop -t 10 nginx
[Install]
WantedBy=multi-user.target
EOF

## Reload systemd config
systemctl daemon-reload

## Enable and start nginx service
systemctl --now enable nginx
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /etc/systemd/system/nginx.service.

## Check the nginx service status
systemctl status nginx
● nginx.service - Nginx Podman container
   Loaded: loaded (/etc/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-02-09 02:52:00 UTC; 4s ago
 Main PID: 9259 (podman)
    Tasks: 8 (limit: 2359)
   Memory: 7.2M
   CGroup: /system.slice/nginx.service
           └─9259 /usr/bin/podman start -a nginx

Feb 09 02:52:00 test.novalocal systemd[1]: Started Nginx Podman container.
{% endhighlight %}
