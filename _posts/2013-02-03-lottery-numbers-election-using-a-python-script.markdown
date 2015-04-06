---
layout: post
status: publish
published: true
title: Python lottery numbers generator
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 184
wordpress_url: http://www.remote-lab.net/?p=184
date: '2013-02-03 13:51:39 +0000'
date_gmt: '2013-02-03 11:51:39 +0000'
categories:
- Python
tags: []
comments:
- id: 6062
  author: Joseph
  author_email: joseph1186@hotmail.com
  author_url: ''
  date: '2013-04-27 02:46:40 +0000'
  date_gmt: '2013-04-27 00:46:40 +0000'
  content: "(random.sample(range(1,50),6))"
- id: 6063
  author: marius
  author_email: marius@remote-lab.net
  author_url: http://www.remote-lab.net/
  date: '2013-04-27 02:50:41 +0000'
  date_gmt: '2013-04-27 00:50:41 +0000'
  content: Nice, thanks for the tip Joseph!
- id: 7172
  author: John
  author_email: jsmith@yahoo.com
  author_url: ''
  date: '2013-06-20 06:28:48 +0000'
  date_gmt: '2013-06-20 04:28:48 +0000'
  content: Great little piece of code, but I would replace the if statement with a
    while loop, because there is a chance that it could generate the same number again.
- id: 8804
  author: Markus
  author_email: hackspacher@gmx.de
  author_url: http://markush.cwsurf.de
  date: '2014-01-09 17:57:17 +0000'
  date_gmt: '2014-01-09 15:57:17 +0000'
  content: "I have make myself a lottogeneator with Qt4 GUI, http://markush.cwsurf.de/joomla_17/index.php/python/pylottosimu/8-lotto-generator-und-simulator\r\n\r\nThe
    webside is in german, but the GUI have I translate in english and six more languages
    and at start looks it for the language code."
- id: 9677
  author: Dazza
  author_email: darrin@pdptech.com.au
  author_url: ''
  date: '2014-03-05 02:55:56 +0000'
  date_gmt: '2014-03-05 00:55:56 +0000'
  content: Thank's joseph, that is an awesome one liner and it does exactly what i
    need.
- id: 16305
  author: alex
  author_email: alexksyed@gmail.com
  author_url: ''
  date: '2014-08-23 19:46:53 +0000'
  date_gmt: '2014-08-23 17:46:53 +0000'
  content: "or to avoid checking for duplicates \r\n\r\nimport random\r\nnumbers =
    []\r\nfor i in range(1,50):\r\n\tnumbers.append(i)\r\n\r\nresult = []\r\nfor i
    in range(6):\r\n\tindex = random.randint(0,49-i)\r\n\tresult.append(numbers[index])\r\n\tdel
    numbers[index]\r\nprint result"
- id: 16306
  author: alex
  author_email: alexksyed@gmail.com
  author_url: ''
  date: '2014-08-23 19:48:43 +0000'
  date_gmt: '2014-08-23 17:48:43 +0000'
  content: "or again with correct formatting (hopefully)\r\n\r\n#!/usr/bin/env python\r\nimport
    random\r\nnumbers = []\r\nfor i in range(1,50):\r\n    numbers.append(i)\r\n\r\nresult
    = []\r\nfor i in range(6):\r\n    index = random.randint(0,49-i)\r\n    result.append(numbers[index])\r\n
    \   del numbers[index]\r\nprint result"
- id: 16307
  author: alex
  author_email: alexksyed@gmail.com
  author_url: ''
  date: '2014-08-23 19:55:41 +0000'
  date_gmt: '2014-08-23 17:55:41 +0000'
  content: "last attempt ... plus fixed the incorrect randint range\r\n\r\n#!/usr/bin/env
    python\r\nimport random\r\nnumbers = []\r\nfor i in range(1,50):\r\n    numbers.append(i)\r\n\r\nresult
    = []\r\nfor i in range(6):\r\n    index = random.randint(0,len(numbers) - 1)\r\n
    \   result.append(numbers[index])\r\n    del numbers[index]\r\nprint result\r\n"
---
<p>Hello guys,</p>
<p>Today I wanted to play a Lotto ticket and as I am currently trying to learn the Python language I thought writing a script that generates the numbers would be a good idea. </p>
<p>The game rules are pretty straight forward: you should choose six numbers from 1 to 49 and if you match all 6 numbers you win the jackpot. With the lottery ticket in our country you can play three sets of numbers so what I need to do in my script is to generate three lists of 6 unique numbers within the 1 to 49 range.</p>
<p><code lang="python[notools]">#!/usr/bin/env python<br />
import random<br />
numbers = []<br />
for i in range(3):<br />
    for j in range(6):<br />
        numbers.append(random.randint(1,49))<br />
        for k in range(j):<br />
            if numbers[j]==numbers[k]:<br />
                numbers[j]=random.randint(1,49)<br />
    print numbers<br />
    numbers = []<br />
</code></p>
<p><?php<br />
include('loto.php');<br />
?></p>
