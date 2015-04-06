---
layout: post
status: publish
published: true
title: Pure-FTPd with MySQL backend
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 176
wordpress_url: http://www.remote-lab.net/?p=176
date: '2013-04-13 23:27:26 +0000'
date_gmt: '2013-04-13 21:27:26 +0000'
categories:
- Linux
tags: []
comments: []
---
<p>This is a quick tutorial on how to install a clean FTP server with MySQL user database backend on a Debian based OS:</p>
<p>Install MySQL Server, Client and Pure-FTPd server:<br />
<code lang="c[notools]">aptitude install mysql-server mysql-client pure-ftpd-mysql</code></p>
<p>Connect to the MySQL server using the local console, create the database, grant privileges for the account the FTP server will use to connect to the DB, create the table that will store the user info and populate it:</p>
<p><code lang="c[notools]">mysql> CREATE DATABASE pureftpd;<br />
mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP ON pureftpd.* TO 'pureftpd'@'localhost' IDENTIFIED BY 'ftpdpass';<br />
mysql> FLUSH PRIVILEGES;<br />
mysql> USE pureftpd;<br />
mysql> CREATE TABLE users (<br />
    -> User varchar(16) NOT NULL default '',<br />
    -> status enum('0','1') NOT NULL default '0',<br />
    -> Password varchar(64) NOT NULL default '',<br />
    -> Uid varchar(11) NOT NULL default '-1',<br />
    -> Gid varchar(11) NOT NULL default '-1',<br />
    -> Dir varchar(128) NOT NULL default '',<br />
    -> comment tinytext NOT NULL,<br />
    -> PRIMARY KEY (User),<br />
    -> UNIQUE KEY User (User)<br />
    -> ) TYPE=MyISAM;<br />
mysql> insert into `users`  (`User`, `status`, `Password`, `Uid`, `Gid`, `Dir`, `comment`) values ('ftpuser', '1', md5('password'), '33', '33', '/var/www/', '');</code></p>
<p>Next, let's edit the config file used by the PureFTP server to connect to the MySQL server and query the database:</p>
<p><code lang="c[notools]">root@remote-lab:~# vim /etc/pure-ftpd/db/mysql.conf<br />
MYSQLSocket      /var/run/mysqld/mysqld.sock<br />
MYSQLServer     localhost<br />
MYSQLPort       3306<br />
MYSQLUser       pureftpd<br />
MYSQLPassword   ftpdpass<br />
MYSQLDatabase   pureftpd<br />
MYSQLCrypt      md5<br />
MYSQLGetPW      SELECT Password FROM users WHERE User="\L" AND status="1"<br />
MYSQLGetUID     SELECT Uid FROM users WHERE User="\L" AND status="1"<br />
MYSQLGetGID     SELECT Gid FROM users WHERE User="\L"AND status="1"<br />
MYSQLGetDir     SELECT Dir FROM users WHERE User="\L"AND status="1"</code></p>
<p>A couple of settings I find useful are to chroot the users in their directories and to set PureFTPd not to resolve hostnames. If you want to add users having an UID lower than 1000 you will also have to change the default config file. This is useful for instance if you add ftp users that need access to directories owned by www-data that has 33 UID. </p>
<p><code lang="c[notools]">echo "yes" > /etc/pure-ftpd/conf/ChrootEveryone<br />
echo "yes" > /etc/pure-ftpd/conf/DontResolve<br />
echo "33" > /etc/pure-ftpd/conf/MinUID</code></p>
<p><code lang="c[notools]">/etc/init.d/pure-ftpd-mysql restart</code></p>
<p>And we should be up and running. </p>
