---
layout: post
status: publish
published: true
title: Dynamic DNS using your own domain
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 179
wordpress_url: http://www.remote-lab.net/?p=179
date: '2013-05-20 13:20:39 +0000'
date_gmt: '2013-05-20 11:20:39 +0000'
categories:
- Linux
tags: []
comments:
- id: 3249
  author: Calin C.
  author_email: calin@firstdigest.com
  author_url: http://firstdigest.com
  date: '2013-05-28 10:15:21 +0000'
  date_gmt: '2013-05-28 08:15:21 +0000'
  content: Excellent "how-to", clear and very easy to follow up. Good job!
- id: 3250
  author: marius
  author_email: marius@remote-lab.net
  author_url: http://www.remote-lab.net/
  date: '2013-05-28 18:21:54 +0000'
  date_gmt: '2013-05-28 16:21:54 +0000'
  content: Thank you for the positive feedback, Calin!
---
<p>I have recently changed my ISP and the new one is providing me an IP address through PPPoE that's changing after every disconnect. </p>
<p>Being in this situation I had two options: set up a dynamic DNS client on my router using a dynamic DNS service such as no-ip.com or use my current DNS server and automatically update its zone records. I decided to go for the latter.</p>
<p>The mechanism I will use is pretty simple: a client inside my LAN will periodically initiate an HTTP request to a particular section of this website that is password protected. The web server I am running remote-lab.net on is also the authoritative name server of remote-lab.net. The server will run a script every couple of minutes that checks the source IP address of completed HTTP requests, it will update the zone file records and increment the zone's serial number. </p>
<p>First, let's install the required servers: Bind9 DNS server and Apache HTTP server:</p>
<p><code lang="c[notools]">aptitude install bind9 apache2</code></p>
<p>By default Apache's default DocumentRoot is placed in /var/www. Let's create a section inside it that is password protected:</p>
<p><code lang="c[notools]">root@remote-lab:~# mkdir /var/www/randomstring<br />
root@remote-lab:~# echo "Dynamic DNS" > /var/www/randomstring/index.html<br />
root@remote-lab:~# vim /var/www/randomstring/.htaccess<br />
AuthUserFile /etc/apache2/htpasswd<br />
AuthName "Please Log In"<br />
AuthType Basic<br />
require user admin<br />
root@remote-lab:~# htpasswd -c /etc/apache2/htpasswd admin<br />
New password:<br />
Re-type new password:<br />
Adding password for user admin<br />
root@remote-lab:~# cat /etc/apache2/htpasswd<br />
admin:NA4V0VYvLWlU2</code></p>
<p>Now let's try a test that sends an HTTP request and see if the server returns a 200 OK. </p>
<p><code lang="bash[notools]">marius@remoteur:~\> curl -s http://admin:pass@www.remote-lab.net/randomstring/index.html</code></p>
<p>Check the apache logs:<br />
<code lang="c[notools]">188.26.130.193, 188.26.130.193 - admin [20/Jan/2013:12:21:39 +0200] "GET /randomstring/index.html HTTP/1.1" 200 23 "-" "curl/7.26.0"</code></p>
<p>Setting up the DNS server. I will use bind9 and show the config files used for remote-lab.net domain as example:</p>
<p>Create the zone file of your domain:</p>
<p><code lang="c[notools]">root@remote-lab:~# vim /etc/bind/db.remote-lab.net</p>
<p>$TTL    120<br />
@   IN  SOA ns.remote-lab.net. marius.remote-lab.net. (<br />
            2012121951  ; Serial<br />
            120         ; Refresh<br />
            86400       ; Retry<br />
            2419200     ; Expire<br />
            120 )   ; Negative Cache TTL<br />
;<br />
@       IN  NS  ns.remote-lab.net.<br />
ns      IN  A   188.240.48.106<br />
@       IN  A   188.240.48.106<br />
@       IN  MX  1 aspmx.l.google.com.<br />
@       IN  MX  5 alt1.aspmx.l.google.com.<br />
@       IN  MX  5 alt2.aspmx.l.google.com.<br />
@       IN  MX  10 aspmx2.googlemail.com.<br />
@       IN  MX  10 aspmx3.googlemail.com.<br />
@       IN  MX  10 aspmx4.googlemail.com.<br />
@       IN  MX  10 aspmx5.googlemail.com.<br />
@       IN  TXT "v=spf1 include:_spf.google.com ip4:188.240.48.106 ~all"<br />
mail    IN  CNAME  ghs.google.com.<br />
www     IN  CNAME  remote-lab.net.<br />
virtual IN  A   188.26.130.193</code></p>
<p>One thing to notice is that I set the TTL value to 3 minutes so that the changes will be quickly propagated. </p>
<p>virtual.remote-lab.net is the hostname assigned to my lab ip address.</p>
<p>Edit the named.conf.local file and add your zone to the config :</p>
<p><code lang="c[notools]">root@remote-lab:~# vim /etc/bind/named.conf.local<br />
zone "remote-lab.net" {<br />
        type master;<br />
        file "/etc/bind/db.remote-lab.net";<br />
};</code></p>
<p><code lang="c[notools]">root@remote-lab:~# /etc/init.d/bind9 restart</code></p>
<p>Restart the server and it should be up and running. You can do a quick test by querying the server for a specific record:</p>
<p><code lang="c[notools]">marius@remoteur:~\> dig @188.240.48.106 virtual.remote-lab.net<br />
;; QUESTION SECTION:<br />
;virtual.remote-lab.net.		IN      	A<br />
;; ANSWER SECTION:<br />
virtual.remote-lab.net.	120     	IN      	A       	188.26.130.193<br />
;; AUTHORITY SECTION:<br />
remote-lab.net.  	        120     IN      	NS      	ns.remote-lab.net.<br />
;; ADDITIONAL SECTION:<br />
ns.remote-lab.net.	120     	IN      	A       	188.240.48.106</code></p>
<p>Create a bash script that checks the web server logs, updates the zone records based on the source IP address of the HTTP requests and sends you an email when a change occurs:</p>
<p><code lang="bash[notools]">#/bin/bash<br />
IPCLIENT=`grep 'randomstring/index.html HTTP/1.1" 200' /var/log/apache2/access_log | tail -1 | awk '{print $2}'`<br />
SERIAL=`grep Serial /etc/bind/db.remote-lab.net | sed -e 's/; Serial//'`<br />
SERIALNEW=$(($SERIAL+1))<br />
IPLAB=`grep virtual /etc/bind/db.remote-lab.net | awk -F" " '{print $4}'`<br />
if [ "$IPCLIENT" == "$IPLAB" ]<br />
then<br />
    :<br />
else<br />
    sed -i "s/$SERIAL/$SERIALNEW/" /etc/bind/db.remote-lab.net<br />
    sed -i "s/$IPLAB/$IPCLIENT/" /etc/bind/db.remote-lab.net<br />
    /etc/init.d/bind9 reload<br />
    echo "The new LAB IP is $IPCLIENT" | mail -s "Your ISP has just changed your assigned IP" -r root@remote-lab.net marius@remote-lab.net<br />
fi</code></p>
<p>The last thing to do is to add our script to the cron jobs and run every several minutes.</p>
<p>Please let me know if you have any questions or if I missed something in this setup.</p>
