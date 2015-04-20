---
layout: post
title: VirtualBox Web Interface
date: '2011-11-22 03:34:53 +0000'
categories:
- Linux
- Virtualization
permalink: virtualbox-web-interface
---
I am personally using <a href="https://www.virtualbox.org/">VirtualBox</a> as a virtualization solution as it fits my needs and it integrates well with Ubuntu. I recently found that you can set up a web interface and manage Virtualbox through it. In the following article I'll describe how you can set up the web interface for VirtualBox using <a href="http://code.google.com/p/phpvirtualbox/">phpVirtualBox</a> and I'd also like to thank the developers for doing this great job.

___

Firstly as with any web interface a web server is required so you'll need to install apache web server with php support :
{% highlight bash %}
root@localhost#apt-get install apache2 php5 libapache2-mod-php5
{% endhighlight %} 

After this you'll have a fully working webserver with php support having its root directory into /var/www/

Now we will need to install VirtualBox. I prefer using the latest version so we'll install VirtualBox 4.1
{% highlight bash %}
root@localhost#apt-get install virtualbox-4.1
{% endhighlight %} 

phpVirtualBox requires vboxwebserv service to run so you'll have to run the following command:
{% highlight bash %}
root@localhost#vboxwebsrv -b --host {ip_address} --port 18083
b: background
host: the ip address of the interface used by the server to listen 
port: the port used  by the server
{% endhighlight %} 

Download phpVirtualBox and move it to you web server's root
{% highlight bash %}
root@localhost#wget http://phpvirtualbox.googlecode.com/files/phpvirtualbox-4.1-5.zip 
root@localhost#unzip  phpvirtualbox-4.1-5.zip 
root@localhost#mv phpvirtualbox-4.1-5 /var/www/vbox 
root@localhost#cd /var/www/vbox
{% endhighlight %} 

Edit the config.php file by modifying the following variables to match your system's:
{% highlight bash %}
root@localhost#vim config.php
 /* Username / Password for system user that runs VirtualBox */
 var $username = ‘vbox’;
 var $password = ‘your-password’;
 var $location = ‘http://{ip_address}:18083/’;
{% endhighlight %} 

And now you should have an accessible web interface for your VirtualBox which can be accessed at http://{ip_address}/vbox. The default username and password are admin/admin.

In order to access the VMs consoles through the web interface you have to install the <a href="http://download.virtualbox.org/virtualbox/4.1.6/Oracle_VM_VirtualBox_Extension_Pack-4.1.6-74713.vbox-extpack">VirtualBox Extension Pack</a> and set the display settings to activate the remote display.

<img class="aligncenter size-full wp-image-93" title="phpvbsm" src="{{'/assets/static/phpvbsm.png' | prepend: site.baseurl | prepend: site.url }}" alt="" width="675" height="500" />
