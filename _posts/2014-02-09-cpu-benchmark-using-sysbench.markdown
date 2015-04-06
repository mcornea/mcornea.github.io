---
layout: post
status: publish
published: true
title: CPU benchmark using sysbench
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 211
wordpress_url: https://remote-lab.net/?p=211
date: '2014-02-09 12:00:22 +0000'
date_gmt: '2014-02-09 10:00:22 +0000'
categories:
- Linux
tags: []
comments: []
---
<p>Sysbench allows you to do a quick performance benchmark of several system parameters such as file I/O, CPU, scheduler and others.</p>
<p>Below are the results for running a 100000 prime numbers calculation using 4 threads on my Intel Core i5-3427U CPU:</p>
<p><code lang="c[notools]">marius@remoteur:~>>> sysbench --num-threads=4 --test=cpu --cpu-max-prime=100000 run<br />
sysbench 0.4.12:  multi-threaded system evaluation benchmark</p>
<p>Running the test with following options:<br />
Number of threads: 4</p>
<p>Doing CPU performance benchmark</p>
<p>Threads started!<br />
Done.</p>
<p>Maximum prime number checked in CPU test: 100000</p>
<p>Test execution summary:<br />
    total time:                          86.6428s<br />
    total number of events:              10000<br />
    total time taken by event execution: 346.5374<br />
    per-request statistics:<br />
         min:                                 33.06ms<br />
         avg:                                 34.65ms<br />
         max:                                158.58ms<br />
         approx.  95 percentile:              37.12ms</p>
<p>Threads fairness:<br />
    events (avg/stddev):           2500.0000/14.25<br />
    execution time (avg/stddev):   86.6344/0.01</code></p>
