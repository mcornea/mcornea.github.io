---
layout: post
title: Graphing car metrics into the cloud with Raspberry Pi, OBD and Graphite  
date: '2015-04-19 19:44:40 +0000'
categories:
- Linux
- Car Automation
permalink: raspberry-pi-car-automation
---

In this post we'll see how you can grab car sensors data and turn them into some good loking and easy to watch graphs.

___

## Motivation:
I've recently changed my car's stock air filter with a performance one which really improved the car's responsiveness. While driving back from Romania to Czech I was thinking that this just can't come for free and I wanted to somehow see how it impacts the car's working parameters. First thing I noticed that changed was the fuel consumption. Since the car board computer only provides instantaneous and average fuel consumption numbers I can't really get to do an analysis of what changed in time. Numbers are good but, at least for me, is hard to interpret them as absolute values and I need to get the whole picture where they stand. The first thought was that turning metrics like rpm, speed and instantaneous fuel consumption into easy to read graphs could get me one step closer to have an insight of how the car is actually behaving.

So there kicked my doer spirit and started thinking of ways of doing it. What I need:

- Data source

I knew most of the cars have a diagnosis port called OBD which is used for reading data from most of the existing sensors in your car.

- Data destination

Once I got all the sensor data I needed to graph it somehow. This would be the easiest task since I've already used Graphite before and knew how is easy is to just pass data and then it did all the graphing stuff.

- Data processor

Now that I have the data source and destination I also needed the device that processes/forwards it. I had a Raspberry Pi standing on my desk for 2 or more years without turning it on so I thought that would be the ideal time to start using it.  

- Put everything together

Basically we have all the system's components but we need to connect them somehow to get functionality out of it. After doing some Google searches I found out that there were some cheap Bluetooth ODB readers on the market and decided to go with that. Since I was going to use Bluetooth for connecting the Raspberry Pi to the ODB reader I also needed a USB Bluetooth dongle.
The last component in the system was getting the Raspberry Pi connected to the Graphite instance. Since I'm a cloudy guy I'm going to run Graphite on my lab Openstack so this means I was going to require an Internet connection on the Pi. First thing that came to mind was getting an USB 3G modem. After doing some reading I found out that most of these modems require external power. I wanted to keep cabling as clean as possible by powering the Pi from the car's USB port so I went for another approach. The solution I came up was to use my phone's tethering capabilities and get the Pi connected to it via WiFi so I also required a USB WiFi dongle.

## Diagram of how this is going to work:
<a href="{{'assets/static/car_odb.png' | prepend: site.baseurl | prepend: site.url }}"><img src="{{'assets/static/car_odb.png' | prepend: site.baseurl | prepend: site.url }}" alt="Car ODB" width="800" height="600"/></a>

## Bill of materials:

- Raspberry Pi Model B
- Kingston 32GB SDHC Memory Card Class 10
- MicroUSB cable
- Asus USB-BT400 Bluetooth dongle
- Edimax EW-7811Un WiFi dongle
- Elm327 OBDII Bluetooth Scanner
- 3G Tethering capable phone

## Before getting started:

- Get your Raspberry Pi installed.
- Make sure it's connected via WiFi to the mobile phone
- Find it's IP address and connect to it via SSH
- Install graphite and grafana. You can use these Ansible roles here: 
  https://github.com/remoteur/ansible-graphs

## TL;DR. Let's get started:

1. Connect the OBDII reader to your car
2. Log in by SSH to the Pi
3. Discover the ODBII mac address or read it from the case:
{% highlight bash %}
hcitool scan
Scanning ...
00:0D:18:00:00:01 OBDII
{% endhighlight %}
4. Add the MAC address to the Bluetooth conf file.
{% highlight bash %}
cat /etc/bluetooth/rfcomm.conf
rfcomm99 {
       bind yes;
       device 00:0D:18:00:00:01;
       channel 1;
       comment "ELM327 based OBD II test tool";
   }
{% endhighlight %}
5. Write init script that will manage the Bluetooth connection through the /dev/rfcomm99 device.
{% highlight bash %}
cat /etc/init.d/elm327
#!/bin/bash
### BEGIN INIT INFO
# Provides: elm327
# Required-Start:
# Required-Stop:
# Should-Start:
# Should-Stop: $null
# Default-Start: 3 5
# Default-Stop: 0 1 2 6
# Short-Description: Start elm327
# Description: starts the elm327 bluetooth device
### END INIT INFO

DevNum=99       # DevNum is depending on the rfcom settings /etc/bluetooth/rfcom.cfg
case $1 in
start)
     rfcomm bind $DevNum
     ;;
stop)
     rfcomm release $DevNum
     ;;
status)
    rfcomm show $DevNum
    ;;
*)
    cat<<EOF
Usage:
    $0 [ start | stop | status ]

EOF
esac
{% endhighlight %}
6. Create the device by running the init script
{% highlight bash %}
sudo /etc/init.d/elm327 start
{% endhighlight %}
7. Install obd python module via pip
{% highlight bash %}
sudo pip install obd
{% endhighlight %}
8. Write a script that reads sensor values and sends them to the graphite instance:
{% highlight python %}
import platform
import socket
import time
import obd

CARBON_SERVER = 'graph.remote-lab.net'
CARBON_PORT = 2003
DELAY = 2  # secs


def send_msg(message):
    #    print 'sending message:\n%s' % message
    sock = socket.socket()
    sock.connect((CARBON_SERVER, CARBON_PORT))
    sock.sendall(message)
    sock.close()

def get_rpms():
    cmd = obd.commands.RPM
    response = connection.query(cmd)
    return response.value

def get_speed():
    cmd = obd.commands.SPEED
    response = connection.query(cmd)
    return response.value

if __name__ == '__main__':
    node = platform.node().replace('.', '-')
    connection = obd.OBD("/dev/rfcomm99")
    while True:
        timestamp = int(time.time())
        rpms = get_rpms()
        speed = get_speed()
        lines = [
            'system.%s.rpm %s %d' % (node, rpms, timestamp),
            'system.%s.speed %s %d' % (node, speed, timestamp),
        ]
        message = '\n'.join(lines) + '\n'
        send_msg(message)
        time.sleep(DELAY)
{% endhighlight %}
9. Run the script and you should start seeing the metrics graphed by Graphite. I'm using Grafana as a Graphite frontend and here's how they look like:
<a href="{{'assets/static/car_stats.png' | prepend: site.baseurl | prepend: site.url }}"><img src="{{'assets/static/car_stats.png' | prepend: site.baseurl | prepend: site.url }}" alt="Car Stats" width="1167" height="657"/></a>
