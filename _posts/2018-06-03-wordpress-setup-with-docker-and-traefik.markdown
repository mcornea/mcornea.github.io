---
layout: post
comments: True
status: publish
published: true
title: Wordpress setup with Docker and Traefik
date: '2018-06-03 10:21:31 +0000'
categories:
- Linux
- Docker
- Traefick
permalink: wordpress-docker-traefik
---

For this weekend project I had to migrate a couple of wordpress websites away from an existing server that I'm going to decommission soon. The current environment consists of a traditional setup with nginx and php-fpm running as systemd services on a CentOS VM with each website being configured within its own server block. This was, of course, an opportunity to come up with a new environment where I could experiment with some components that I've recently learned about.

___

So let's go ahead and put down the requirements for this new environment and see what I can come up with:

1. Availability
  * The system does not need to be highly available, this is a single node environment. Since the workloads are mostly personal blogs of my lovely wife and other relatives it’s acceptable that in the eventuality the system goes kaput the websites can stay down until I manually recover from backups.

2. Reproducibility
  * It’s important that the new environment is easy to reproduce. This is related to 1/ as in case the server disappears for whatever reason I need to be able to quickly bring up the same environment on a different machine and recover the databases and document root files from backups.

3. Experimentation
  * Last but not least this is an environment where I want to play and experiment. First I want to try out <a href="https://traefik.io/" target="_blank">Træfik</a> which is a new load balancer and reverse proxy that I’ve read about few days ago and looks promising. Second I want to practice the containerized way of applications packaging and see how it fits an operational workflow(keeping the system up to date, applying patches, backup/restore, etc). Since the workloads that are running on the system are not mission critical but in the same time I cannot afford to get my wife upset by keeping her blog unavailable for days this is the perfect environment where I can learn and get some skin in the game.

___

Enough with the talk now let’s move to actions and see how to set up the environment.

* Get the cheapest VM from <a href="https://www.hetzner.com/cloud" target="_blank">Hetzner</a>
  * 3EUR/month for 1vCPU, 2GB RAM, 20GB disk space(on local SSD). I’ll use CentOS as OS but the instructions here(except installing the docker package) should be pretty much the same as the magic happens inside the container.

* Once the server gets online install Docker per the <a href="https://docs.docker.com/install/linux/docker-ce/centos/" target="_blank">official docs</a>

* Prepare containers persistent storage directories. 
  * We want the application data, in our case the wordpress files and the databases, to be stored on a persistent location so we use the bind mount feature to expose host directories inside the containers.
{% highlight bash %}
mkdir -p /storage/{db,www}
{% endhighlight %}

* Create a docker network for the internal containers communication.
{% highlight bash %}
docker network create internal
{% endhighlight %}

* At this point we’re ready to start creating the containers. 

* Prepare the Traefik container configuration file.

{% highlight bash %}
cat << EOF > $PWD/traefik.toml
debug = false
checkNewVersion = true
logLevel = "ERROR"

[web]
address = ":8080"
  [web.auth.basic]
## note: replace password with proper
## generated hash via htpasswd
  users = ["admin:password"]
[entryPoints]
  [entryPoints.http]
  address = ":80"

# Enable more detailed statistics.
[web.statistics]
# Number of recent errors logged.
recentErrors = 10
EOF
{% endhighlight %}

* Create the Traefik loadbalancer container

{% highlight bash %}
docker run -d \
  --restart always \
  --volume /var/run/docker.sock:/var/run/docker.sock \
  --volume $PWD/traefik.toml:/traefik.toml \
  --publish 80:80 \
  --label traefik.frontend.rule=Host:lb00.remote-lab.net \
  --label traefik.frontend.redirect.regex='^http://lb00.remote-lab.net/(.*)' \
  --label traefik.frontend.redirect.replacement='https://lb00.remote-lab.net/$1' \
  --label traefik.port=8080 \
  --network internal \
  --name traefik \
  traefik:latest --docker
{% endhighlight %}

* Create the MariaDB database container

{% highlight bash %}
docker run -d \
  --name mariadb \
  --restart always \
  --volume /storage/db:/var/lib/mysql \
  --network internal \
  -e MYSQL_ROOT_PASSWORD=$(openssl rand -hex 20) \
  mariadb:latest
{% endhighlight %}

* Create wordpress00 website
{% highlight bash %}
mkdir /storage/www/wordpress00.remote-lab.net
docker run --name wordpress00.remote-lab.net \
  --restart always \
  --label="traefik.backend=wordpress00.remote-lab.net" \
  --label="traefik.frontend.rule=Host:wordpress00.remote-lab.net,www.wordpress00.remote-lab.net" \
  --label="traefik.docker.network=internal" \
  --label="traefik.port=80" \
  --network internal \
  --link mariadb:db \
  -v /storage/www/wordpress00.remote-lab.net:/var/www/html \
  -d wordpress:latest
{% endhighlight %}

* Create wordpress01 website
{% highlight bash %}
mkdir /storage/www/wordpress01.remote-lab.net
docker run --name wordpress01.remote-lab.net \
  --restart always \
  --label="traefik.backend=wordpress01.remote-lab.net" \
  --label="traefik.frontend.rule=Host:wordpress01.remote-lab.net,www.wordpress01.remote-lab.net" \
  --label="traefik.docker.network=internal" \
  --label="traefik.port=80" \
  --network internal \
  --link mariadb:db \
  -v /storage/www/wordpress01.remote-lab.net:/var/www/html \
  -d wordpress:latest
{% endhighlight %}

* This is a snapshot of the Traekif loadbalancer dashboard. All that I had to do is to create the toml configuration file and run the container. It's quite awesome given that all you need to do when creating a new wordpress container is to pass the specific labels and it gets autodetected and configured by Traefik. Pretty cool stuff.

<a href="{{'/public/images/traefik.png' | prepend: site.baseurl | prepend: site.url }}"><img src="{{'/public/images/traefik.png' | prepend: site.baseurl | prepend: site.url }}" alt="Traefik Dashboard" width="800" height="480"/></a>


* Note: at this point the wordpress containers run uninitialized Wordpress installations. I won't cover the restore part from my current environment in this post to keep it simple for now.

* I am using Cloudflare for CDN and hosting the DNS zone for my domain. At this point I am ready to set the DNS records and point the websites to this new server. Cloudflare does SSL termination by default so I'm configuring http to https redirection from inside the Wordpress application. 

___

Summary
--------

The entire process has been extremely fast and I love that. You can see in the recording below that even if the commands were run manually the entire procedure took just few minutes. I know that in the end the complexity doesn't disappear but instead it's just moved to the Dockerfiles. I have split feeling in regards to this - on one hand I'm not feeling comfortable about not knowing what's going inside the container but on the other hand I feel that an entire community around a certain project can do a great job at maintaining the Dockerfile. One thing is for sure - containers are such a convenient way of shipping the application.

<a href="https://traefik.io/" target="_blank">Træfik.</a> I got a quick glimpse at its capabilities and I like it a lot. I am curious to dig deeper into its configuration details and more complex deployments such as HA or Kubernetes integration.

I can declare this a productive Sunday. Feel free to reach out if you have any questions!

* Below is a recording of the entire process

<head>
  <link rel="stylesheet" type="text/css" href="{{'/public/asciinema-player.css' | prepend: site.baseurl | prepend: site.url }}" />
</head>
<body>
  <asciinema-player src="{{ '/public/images/wordpress.cast' | prepend: site.baseurl | prepend: site.url }}" idle_time_limit="1" rows="40"></asciinema-player>
  <script src="{{ '/public/asciinema-player.js' | prepend: site.baseurl | prepend: site.url }}"></script>
</body>
