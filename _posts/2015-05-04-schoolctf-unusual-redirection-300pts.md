---
id: 579
title: 'SchoolCTF : Unusual redirection 300pts'
date: 2015-05-04T01:16:03+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=579
permalink: /2015/05/schoolctf-unusual-redirection-300pts/
category: Web Exploitation
tags: [SchoolCTF]
---
> ### Unusual redirection
> 
> Somebody has moved the flag so it is not available to everyone, but old link still is usefull <a href="http://sibears.ru:11611/" target="_blank">http://sibears.ru:11611/</a>

<!--more-->

flag yang lama berada di http://sibears.ru:11611/flag tapi di redirect ke http://sibears.ru:11611/oops jadi isi dari flag tidak bisa keliatan , sementara file flag masih di temukan

<div id="attachment_580" style="width: 1336px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.github.io/images/2015/05/oldflag.png"><img class="size-full wp-image-580" src="http://abdilahrf.github.io/images/2015/05/oldflag.png" alt="SchoolCTF : Unusual Redirect" width="1326" height="226" /></a>
  
  <p class="wp-caption-text">
    Flag lama
  </p>
</div>

coba untuk melihat isi dari flag dengan menggunakan &#8220;view-source:sibears.ru:11611/flag&#8221;  untuk google chrome , ternyata tetap terkena direct juga, mungkin menggunakan curl bisa tidak mengikuti direct tersebut tanpa option &#8211;location

<div id="attachment_581" style="width: 681px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.github.io/images/2015/05/flag7.png"><img class="size-full wp-image-581" src="http://abdilahrf.github.io/images/2015/05/flag7.png" alt="SchoolCTF : Unusual Redirect" width="671" height="119" /></a>
  
</div>

Flag : **old\_link\_is\_cool\_link**

&nbsp;