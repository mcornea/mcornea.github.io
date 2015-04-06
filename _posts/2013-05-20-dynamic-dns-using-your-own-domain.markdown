---
layout: post
title: Dynamic DNS using your own domain
date: '2013-05-20 13:20:39 +0000'
categories:
- Linux
- Automation
permalink: dynamic-dns-using-your-own-domain
---
I have recently changed my ISP and the new one is providing me an IP address through PPPoE that's changing after every disconnect. 
Being in this situation I had two options: set up a dynamic DNS client on my router using a dynamic DNS service such as no-ip.com or use my current DNS server and automatically update its zone records. I decided to go for the latter.

___

The mechanism I will use is pretty simple: a client inside my LAN will periodically initiate an HTTP request to a particular section of this website that is password protected. The web server I am running remote-lab.net on is also the authoritative name server of remote-lab.net. The server will run a script every couple of minutes that checks the source IP address of completed HTTP requests, it will update the zone file records and increment the zone's serial number. 

First, let's install the required servers: Bind9 DNS server and Apache HTTP server:

{% highlight bash %}
aptitude install bind9 apache2
{% endhighlight %} 

By default Apache's default DocumentRoot is placed in /var/www. Let's create a section inside it that is password protected:
{% highlight bash %}
root@remote-lab:~# mkdir /var/www/randomstring
root@remote-lab:~# echo "Dynamic DNS" > /var/www/randomstring/index.html
root@remote-lab:~# vim /var/www/randomstring/.htaccess
AuthUserFile /etc/apache2/htpasswd
AuthName "Please Log In"
AuthType Basic
require user admin
root@remote-lab:~# htpasswd -c /etc/apache2/htpasswd admin
New password:
Re-type new password:
Adding password for user admin
root@remote-lab:~# cat /etc/apache2/htpasswd
admin:NA4V0VYvLWlU2
{% endhighlight %} 

Now let's try a test that sends an HTTP request and see if the server returns a 200 OK. 
{% highlight bash %}
marius@remoteur:~\> curl -s http://admin:pass@www.remote-lab.net/randomstring/index.html
{% endhighlight %} 

Check the apache logs:
{% highlight bash %}
188.26.130.193, 188.26.130.193 - admin [20/Jan/2013:12:21:39 +0200] "GET /randomstring/index.html HTTP/1.1" 200 23 "-" "curl/7.26.0"
{% endhighlight %} 

Setting up the DNS server. I will use bind9 and show the config files used for remote-lab.net domain as example:
Create the zone file of your domain:

{% highlight bash %}
root@remote-lab:~# vim /etc/bind/db.remote-lab.net
$TTL    120
@   IN  SOA ns.remote-lab.net. marius.remote-lab.net. (
            2012121951  ; Serial
            120         ; Refresh
            86400       ; Retry
            2419200     ; Expire
            120 )   ; Negative Cache TTL
;
@       IN  NS  ns.remote-lab.net.
ns      IN  A   188.240.48.106
@       IN  A   188.240.48.106
@       IN  MX  1 aspmx.l.google.com.
@       IN  MX  5 alt1.aspmx.l.google.com.
@       IN  MX  5 alt2.aspmx.l.google.com.
@       IN  MX  10 aspmx2.googlemail.com.
@       IN  MX  10 aspmx3.googlemail.com.
@       IN  MX  10 aspmx4.googlemail.com.
@       IN  MX  10 aspmx5.googlemail.com.
@       IN  TXT "v=spf1 include:_spf.google.com ip4:188.240.48.106 ~all"
mail    IN  CNAME  ghs.google.com.
www     IN  CNAME  remote-lab.net.
virtual IN  A   188.26.130.193
{% endhighlight %} 

One thing to notice is that I set the TTL value to 3 minutes so that the changes will be quickly propagated. 
virtual.remote-lab.net is the hostname assigned to my lab ip address.
Edit the named.conf.local file and add your zone to the config :

{% highlight bash %}
root@remote-lab:~# vim /etc/bind/named.conf.local
zone "remote-lab.net" {
        type master;
        file "/etc/bind/db.remote-lab.net";
};
{% endhighlight %} 

{% highlight bash %}
root@remote-lab:~# /etc/init.d/bind9 restart
{% endhighlight %} 

Restart the server and it should be up and running. You can do a quick test by querying the server for a specific record:
{% highlight bash %}
marius@remoteur:~\> dig @188.240.48.106 virtual.remote-lab.net
;; QUESTION SECTION:
;virtual.remote-lab.net.		IN      	A
;; ANSWER SECTION:
virtual.remote-lab.net.	120     	IN      	A       	188.26.130.193
;; AUTHORITY SECTION:
remote-lab.net.  	        120     IN      	NS      	ns.remote-lab.net.
;; ADDITIONAL SECTION:
ns.remote-lab.net.	120     	IN      	A       	188.240.48.106
{% endhighlight %} 

Create a bash script that checks the web server logs, updates the zone records based on the source IP address of the HTTP requests and sends you an email when a change occurs:
{% highlight bash %}
#!/bin/bash
IPCLIENT=`grep 'randomstring/index.html HTTP/1.1" 200' /var/log/apache2/access_log | tail -1 | awk '{print $2}'`
SERIAL=`grep Serial /etc/bind/db.remote-lab.net | sed -e 's/; Serial//'`
SERIALNEW=$(($SERIAL+1))
IPLAB=`grep virtual /etc/bind/db.remote-lab.net | awk -F" " '{print $4}'`
if [ "$IPCLIENT" == "$IPLAB" ]
then
    :
else
    sed -i "s/$SERIAL/$SERIALNEW/" /etc/bind/db.remote-lab.net
    sed -i "s/$IPLAB/$IPCLIENT/" /etc/bind/db.remote-lab.net
    /etc/init.d/bind9 reload
    echo "The new LAB IP is $IPCLIENT" | mail -s "Your ISP has just changed your assigned IP" -r root@remote-lab.net marius@remote-lab.net

fi
{% endhighlight %} 

The last thing to do is to add our script to the cron jobs and run every several minutes.

Please let me know if you have any questions or if I missed something in this setup.
