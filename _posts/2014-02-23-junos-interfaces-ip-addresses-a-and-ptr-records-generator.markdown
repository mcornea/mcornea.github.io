---
layout: post
status: publish
published: true
title: JunOS interfaces IP addresses DNS records generator
date: '2014-02-23 21:28:17 +0000'
categories:
- Linux
- Automation
permalink: /junos-interfaces-ip-addresses-dns-records-generator
---
This post is closely related to the previous one where I showed how you can parse the interfaces IP addresses from a curly bracket JunOS config file. The following script will be used to generate A and PTR records for a BIND zone file. Please note that the script needs to be run within the same directory as the Perl parser script and the config file.

___

The entries will have the following format:

{% highlight bash %}
type-fpc-pic-port 300 IN A $value
$value IN PTR type-fpc-pic-port.hostname.domain
{% endhighlight %} 

{% highlight bash %}
#/bin/bash
PERL='/usr/bin/perl'
PARSER='./parser.pl'
CONFIG_FILE='config.txt'
CONFIG_SYS='sys'
CONFIG_INT='int'
hostname=`$PERL $PARSER $CONFIG_SYS | grep host-name | sed -e s/host-name// -e s/\;// | tr '\r' ' ' | sed -e s/\ //g`
domain=`$PERL $PARSER $CONFIG_SYS | grep domain-name | sed -e s/domain-name// -e s/\;// | tr '\r' ' ' | sed -e s/\ //g`
for i in `grep "ge-[0-9]\/[0-9]\/[0-9] {\|ae[0-9] {\|lo[0-9] {" $CONFIG_FILE | sed -e s/\ //g  -e s/\{// | tr '\r' ' '`
    do
        intname=`echo $i | sed s/\\\//-/g`
        for j in `$PERL $PARSER $CONFIG_INT $i | grep unit | sed -e s/\ unit//g  -e s/\{// -e s/\ //g | tr '\r' ' '`
            do
                inetaddr=`$PERL $PARSER $CONFIG_INT $i $j | tr '\r' ' '`
                lastoct=`echo $inetaddr | awk -F '.' {'print $4'}`
                if [ $j -eq 0 ]
                then
                    echo "$intname 300 IN A  $inetaddr"
                    echo "$lastoct  IN  PTR $intname.$hostname.$domain."
                    echo</p>
                else
                    if [ $j -eq 10 ]
                    then
                        echo "$intname 300 IN A  $inetaddr"
                        echo "$lastoct  IN  PTR $intname.$hostname.$domain."
                        echo
                    else
                        echo "$intname-u$j 300 IN A  $inetaddr"
                        echo "$lastoct  IN  PTR $intname-u$j.$hostname.$domain."
                        echo
                    fi
                fi
            done
    done
{% endhighlight %} 
