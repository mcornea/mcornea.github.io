---
layout: post
comments: True
title: CPU benchmark using sysbench
date: '2014-02-09 12:00:22 +0000'
categories:
- Linux
- Benchmark
permalink: cpu-benchmark-using-sysbench
---
Sysbench allows you to do a quick performance benchmark of several system parameters such as file I/O, CPU, scheduler and others.
Below are the results for running a 100000 prime numbers calculation using 4 threads on my Intel Core i5-3427U CPU:

___

{% highlight bash %}
marius@remoteur:~>>> sysbench --num-threads=4 --test=cpu --cpu-max-prime=100000 run
sysbench 0.4.12:  multi-threaded system evaluation benchmark
Running the test with following options:
Number of threads: 4
Doing CPU performance benchmark
Threads started!
Done.
Maximum prime number checked in CPU test: 100000
Test execution summary:
    total time:                          86.6428s
    total number of events:              10000
    total time taken by event execution: 346.5374
    per-request statistics:
         min:                                 33.06ms
         avg:                                 34.65ms
         max:                                158.58ms
         approx.  95 percentile:              37.12ms
Threads fairness:
    events (avg/stddev):           2500.0000/14.25
    execution time (avg/stddev):   86.6344/0.01</code>
{% endhighlight %} 
