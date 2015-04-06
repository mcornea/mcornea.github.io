---
layout: post
title: Pure-FTPd with MySQL backend
date: '2013-04-13 23:27:26 +0000'
categories:
- Linux
permalink: pure-ftpd-with-mysql-backend
---
This is a quick tutorial on how to install a clean FTP server with MySQL user database backend on a Debian based OS:

___

Install MySQL Server, Client and Pure-FTPd server:
{% highlight bash %}
aptitude install mysql-server mysql-client pure-ftpd-mysql
{% endhighlight %} 

Connect to the MySQL server using the local console, create the database, grant privileges for the account the FTP server will use to connect to the DB, create the table that will store the user info and populate it:
{% highlight bash %}
mysql> CREATE DATABASE pureftpd;
mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP ON pureftpd.* TO 'pureftpd'@'localhost' IDENTIFIED BY 'ftpdpass';
mysql> FLUSH PRIVILEGES;
mysql> USE pureftpd;
mysql> CREATE TABLE users (
    -> User varchar(16) NOT NULL default '',
    -> status enum('0','1') NOT NULL default '0',
    -> Password varchar(64) NOT NULL default '',
    -> Uid varchar(11) NOT NULL default '-1',
    -> Gid varchar(11) NOT NULL default '-1',
    -> Dir varchar(128) NOT NULL default '',
    -> comment tinytext NOT NULL,
    -> PRIMARY KEY (User),
    -> UNIQUE KEY User (User)
    -> ) TYPE=MyISAM;
mysql> insert into `users`  (`User`, `status`, `Password`, `Uid`, `Gid`, `Dir`, `comment`) values ('ftpuser', '1', md5('password'), '33', '33', '/var/www/', '');
{% endhighlight %} 

Next, let's edit the config file used by the PureFTP server to connect to the MySQL server and query the database:
{% highlight bash %}
root@remote-lab:~# vim /etc/pure-ftpd/db/mysql.conf
MYSQLSocket      /var/run/mysqld/mysqld.sock
MYSQLServer     localhost
MYSQLPort       3306
MYSQLUser       pureftpd
MYSQLPassword   ftpdpass
MYSQLDatabase   pureftpd
MYSQLCrypt      md5
MYSQLGetPW      SELECT Password FROM users WHERE User="\L" AND status="1"
MYSQLGetUID     SELECT Uid FROM users WHERE User="\L" AND status="1"
MYSQLGetGID     SELECT Gid FROM users WHERE User="\L"AND status="1"
MYSQLGetDir     SELECT Dir FROM users WHERE User="\L"AND status="1"
{% endhighlight %} 

A couple of settings I find useful are to chroot the users in their directories and to set PureFTPd not to resolve hostnames. If you want to add users having an UID lower than 1000 you will also have to change the default config file. This is useful for instance if you add ftp users that need access to directories owned by www-data that has 33 UID. 

{% highlight bash %}
echo "yes" > /etc/pure-ftpd/conf/ChrootEveryone
echo "yes" > /etc/pure-ftpd/conf/DontResolve
echo "33" > /etc/pure-ftpd/conf/MinUID
{% endhighlight %} 
{% highlight bash %}
/etc/init.d/pure-ftpd-mysql restart
{% endhighlight %} 

And we should be up and running. 
