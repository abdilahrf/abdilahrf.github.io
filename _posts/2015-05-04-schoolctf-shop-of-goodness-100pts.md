---
id: 572
title: 'SchoolCTF 2014 : Shop of goodness 100pts'
date: 2015-05-04T00:59:49+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=572
permalink: /2015/05/schoolctf-shop-of-goodness-100pts/
category: CTF
tags: [Web Exploitation]
---
> ### SchoolCTF : Shop of goodness
> 
> Let&#8217;s buy that awesome flag! http://sibears.ru:11011/

<!--more-->

websitenya ternyata menggunakan bahasa rusia jadi kita translate , kebetulan di google chrome autotranslate jadi bahasa ingris

<div id="attachment_573" style="width: 1009px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.github.io/images/2015/05/home.png"><img class="size-full wp-image-573" src="http://abdilahrf.github.io/images/2015/05/home.png" alt="SchoolCTF : Shop of goodness" width="999" height="545" /></a>
  
  <p class="wp-caption-text">
    Homepage
  </p>
</div>

tujuannya di sini kita harus membeli flag , tapi ketika saat mengisi form kita harus mengisi sesuai dengan yang ada di database , pertama kita memikirkan ada kemungkinan sql injection tapi ternyata bukan itu

setelah diperhatikan ternyata variabel &#8220;st&#8221; mewakili step di page tersebut karena kita stuck di page 4 , karena dia menggunakan method **get** jadi kita bisa meloncati step nya jadi 5

http://sibears.ru:11011/buy.php?tid=1&st=5

<div id="attachment_574" style="width: 1007px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.github.io/images/2015/05/flag5.png"><img class="size-full wp-image-574" src="http://abdilahrf.github.io/images/2015/05/flag5.png" alt="SchoolCTF : Shop of goodness" width="997" height="343" /></a>
  
  <p class="wp-caption-text">
    Flag
  </p>
</div>

Flag : **GoOgle\_iS\_yOur_FrieNd**