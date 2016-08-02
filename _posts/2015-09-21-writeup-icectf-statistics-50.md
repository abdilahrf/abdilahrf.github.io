---
id: 1873
title: 'writeup IceCTF Statistics &#8211; 50'
date: 2015-09-21T22:47:35+00:00
author: abdilahrf
layout: post
guid: https://www.hasnydes.us/?p=1873
permalink: /2015/09/writeup-icectf-statistics-50/
factory_shortcodes_assets:
  - 'a:0:{}'
categories:
  - ice ctf
---

> IceCTF Statistics &#8211; 50
>   **Programming**
>   Daniel is strangely good at computing statistics in his head, so instead of a password, a program asks him a series of statistics questions for authentication. Let&#8217;s show him how insecure that is. You can access the server with  <code>nc vuln2015.icec.tf 9000</code>.

  ceritanya di sini daniel login tanpa menggunakan password , tapi menggunakan pertanyaan2 , kalau di tidak salah ini mirip juga dengan problem <a href="https://github.com/ctfs/write-ups-2014/tree/master/seccon-ctf-2014/choose-the-number">chose the number seccon</a> bedanya di sini ada sum dan average, dengan bantuan dari om <a href="https://www.facebook.com/muhammad.abrari?fref=ts">Muhammad abrar istiadi</a> , kami menggunakan <a href="https://github.com/Gallopsled/pwntools">pwntools</a> untuk menyelesaikan problem ini dengan python


    
```python 
    from pwn import *
    import numpy
    
    r = remote("vuln2015.icec.tf", 9000)
    
    i = 1
    
    for a in range(0, 200):
    
        recv = r.recvuntil("the numbers:")
    
        recv = recv.split("\n")
    
        num = recv[0].strip()
        soal = recv[1]
    
        print "[receiving] " + num
        print "[receiving] " + soal
    
        num = [int(x) for x in num.split(" ")]
    
        answer = 0
    
        if "maximum" in soal:
            answer = max(num)
        elif "minimum" in soal:
            answer = min(num)
        elif "average" in soal:
            answer = numpy.average(num)
        elif "sum" in soal:
            answer = numpy.sum(num)
    
        print "[answer] " + str(answer)
        print "[count] " + str(i)
    
        i += 1
    
        r.send(str(answer) + "\n")
    
    print r.recv(4096)
    print r.recv(4096)
    print r.recv(4096)
    print r.recv(4096)```
    

[<img class="aligncenter size-large wp-image-1876" src="http://abdilahrf.github.io/images/2015/09/statistic-1024x576.png" alt="statistic" width="1024" height="576" />](http://abdilahrf.github.io/images/2015/09/statistic.png)