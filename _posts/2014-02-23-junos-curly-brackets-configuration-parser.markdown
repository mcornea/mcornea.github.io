---
layout: post
title: JunOS config interfaces IP address parser
date: '2014-02-23 20:58:36 +0000'
categories:
- Linux
- Automation
permalink: junos-curly-brackets-config-parser
---
In this post I will show how you can obtain an interface IP address out of a JunOS curly brackets configuration file. You may find below the script and also the source configuration file. Please note that in order to run the script both files need to be placed in the same directory. 

___

Please check it and let me know what you think, it's my first Perl script so it could definitely be improved.

{% highlight perl %}
#!/usr/bin/perl
open (CONFIG, 'config.txt');
my $data = do { local $/; <CONFIG>;};
close (CONFIG);
my $system = $data =~ m{(\bsystem\s*({(?:(?>[^{}]+)|(?-1))*}))}
    ? $1
    : die "system not found";
my $intconfig = $data =~ m{(\binterfaces\s*({(?:(?>[^{}]+)|(?-1))*}))}
    ? $1
    : die "interfaces not found";
if ($ARGV[0] eq 'sys') {
    print $system;
}
if ($ARGV[0] eq 'int') {
    if (!defined $ARGV[1]) {
        print $intconfig, "\n";
    }
    if (defined $ARGV[1]) {
        my $int = $intconfig =~ m{(\b$ARGV[1]\s*({(?:(?>[^{}]+)|(?-1))*}))}
            ? $1
            : die "$ARGV[1] not found";
        if (!defined $ARGV[2]) {
            print $int. "\n";
        }
        if (defined $ARGV[2]) {
            my $unit = $int =~m{(\bunit $ARGV[2]\s*({(?:(?>[^{}]+)|(?-1))*}))}
                ? $1
                : die "$ARGV[2] not found";
            my $inet = $unit =~ m{(\bfamily inet\s*({(?:(?>[^{}]+)|(?-1))*}))}
                ? $1
                : die "family inet not found in section";
            my $inetaddr = $inet =~ m{\baddress\s(\d{1,3}(?:\.\d{1,3}){3})}
                ? $1
                : die "no IP address";
            print $inetaddr, "\n";
        }
    }
}
if ($ARGV[0] eq '--help' or !defined $ARGV[0]) {
    print "Usage : ./parser.pl sys                              # outputs system section config", "\n";
    print "        ./parser.pl int                              # outputs interfaces section config", "\n";
    print "        ./parser.pl int [int-name]                   # outputs specific interface section config", "\n";
    print "        ./parser.pl int ge-1/1/7                     # outputs ge-1/1/7 interface section config", "\n";
    print "        ./parser.pl int [int-name] [unit-id]         # outputs specific interface unit IP address", "\n";
    print "        ./parser.pl int ge-1/1/7 1001                # outputs ge-1/1/7 interface unit 1001 IP address", "\n";
}
</code>
<code lang="perl[notools]">
marius@remoteur:~>>> cat config.txt
system {
    host-name junos-device;
    domain-name corporate.net
    time-zone Europe/Bucharest;
    default-address-selection;
    no-redirects;
    location country-code RO;
}
interfaces {
    ge-1/0/0 {
        description "Core: R:core1 RP:ge-0/1/4 (ptp, isis)";
        mtu 9192;
        unit 0 {
            family inet {
                address 192.168.140.29/31;
            }
        }
    }
    ge-1/0/1 {
        description "Cust: R:cust-a RP:ge-1/0/0 (srx240H)";
        unit 0 {
            family inet {
                address 172.16.166.196/30;
            }
        }
    }
    ge-1/0/2 {
        flexible-vlan-tagging;
        native-vlan-id 10;
        mtu 9192;
        unit 10 {
            description "Cust: R:cust-b (data, feed A)";
            vlan-id 10;
            family inet {
                address 192.168.136.184/31;
            }
        }
        unit 1001 {
            description "Core: R:cust-b (cpe management)";
            vlan-id 1001;
            family inet {
                filter {
                    output Protect-cpe;
                }
                address 10.15.4.6/30;
            }
        }
    }
    ae0 {
        description "Core: R:colo-vc2 RI:ae5";
        aggregated-ether-options {
            minimum-links 1;
            link-speed 1g;
        }
        unit 0 {
            family inet {
                address 192.168.140.126/31;
            }
        }
    }
    lo0 {
        unit 0 {
            description "Core: R:primary routing loopback";
            family inet {
                address 192.168.128.166/32;
            }
            }
        }
    }
}
{% endhighlight %} 
