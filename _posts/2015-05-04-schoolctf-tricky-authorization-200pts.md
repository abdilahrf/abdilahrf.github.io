---
id: 558
title: 'SchoolCTF 2014 : Tricky Authorization 200pts'
date: 2015-05-04T00:22:59+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=558
permalink: /2015/05/schoolctf-tricky-authorization-200pts/
category: CTF
tags: [Forensic]
---
> ### SchoolCTF : Tricky Authorization 200pts
> 
> There is some tricky authorization, will you [take a look](http://school-ctf.org/files/task_35f2cc9bcf7e8d5ca53bb29764f323ca32aa14e8.pcap)?
> 
> `nc sibears.ru 11811`

<!--more-->

tipe soal berbentuk file .pcap kebetulan datanya tidak terlalu banyak jadi tidak terlalu sulit menemukan stream yang di cari

[<img class="aligncenter size-full wp-image-559" src="http://abdilahrf.github.io/images/2015/05/authorization.png" alt="SchoolCTF : Tricky Authorization" width="1366" height="768" />](http://abdilahrf.github.io/images/2015/05/authorization.png)

yang biru adalah dari server yang merah adalah response dari user dan termasuk kunci authorization nya , kita coba ikuti connect ke nc sibears.ru 11811

<div id="attachment_560" style="width: 689px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.github.io/images/2015/05/flag2.png"><img class="size-full wp-image-560" src="http://abdilahrf.github.io/images/2015/05/flag2.png" alt="SchoolCTF : Tricky Authorization" width="679" height="417" /></a>
  
  <p class="wp-caption-text">
    Flag
  </p>
</div>

FLAG: **p1ain\_tex7\_is\_sti11\_p1ain\_ev3n\_if\_its\_tricky
  
** 

&nbsp;