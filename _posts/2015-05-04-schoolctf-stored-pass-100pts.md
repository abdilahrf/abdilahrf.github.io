---
id: 576
title: 'SchoolCTF : Stored Pass 100pts'
date: 2015-05-04T01:04:55+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=576
permalink: /2015/05/schoolctf-stored-pass-100pts/
categories: Web Exploitation
tags: [SchoolCTF]
---
> ### SchoolCTF : Stored pass
> 
> Hmm, that password seems to be stored in the browser, but I sure that I have never visited that site <a href="http://sibears.ru:11411/" target="_blank">http://sibears.ru:11411/</a>

<!--more-->

seteleah di buka websitenya menampilkan halaman login dengan password yang sudah tersimpan , dengan inspect element kita bisa membuat input type=&#8221;password&#8221; menjadi input type=&#8221;text&#8221; agar bisa di baca dan menemukan flagnya

<div id="attachment_577" style="width: 929px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.me/images/2015/05/flag6.png"><img class="size-full wp-image-577" src="http://abdilahrf.me/images/2015/05/flag6.png" alt="SchoolCTF : Stored Pass" width="919" height="575" /></a>
  
  <p class="wp-caption-text">
    Flag
  </p>
</div>

Flag : **saving\_passwords\_is\_bad\_idea**