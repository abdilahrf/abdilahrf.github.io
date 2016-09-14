---
id: 191
title: 'Bermain Dengan Bash #1'
date: 2014-07-17T16:14:11+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=191
permalink: /2014/07/bermain-dengan-bash-1/
factory_shortcodes_assets:
  - 'a:0:{}'
categories:
  - Bash
---
Kali ini coba kita bermain dengan bash , cara grab domain dan subdomain dari file html
  
siapkan 1 file .html yang memiliki link ke berbagai sub domain seperti
  
href=&#8221;example.com&#8221; , href=&#8221;sub.example.com&#8221;

kita coba memulai dengan mengambil isi seluruh dari file.html

<div id="attachment_192" style="width: 820px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.me/images/2014/07/terminal.png"><img class="size-full wp-image-192" src="http://abdilahrf.me/images/2014/07/terminal.png" alt="terminal" width="810" height="461" /></a>
  
  <p class="wp-caption-text">
    terminal
  </p>
</div>

<!--more-->

ketika kita menggunakan perintah cat maka semua text yang ada di dalam file tersebut akan di tampilkan ke layar kita
  
bagaimana caranya jika kita ingin mengambil nama domainnya saja

kita coba ambil yang hanya memiliki href=
  
dengan command **grep &#8220;href=&#8221; ghost.html**

<div id="attachment_195" style="width: 708px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.me/images/2014/07/grep-href.png"><img class="size-full wp-image-195" src="http://abdilahrf.me/images/2014/07/grep-href.png" alt="grep href" width="698" height="465" /></a>
  
  <p class="wp-caption-text">
    grep href
  </p>
</div>

masih ada tag html yang sebenarnya kita tidak perlukan muncul ke layar
  
dan kita bisa gunakan fungsi cut delimeter menggunakan tanda / untuk mengambil element ke 3
  
karena setiap domain yang di awali dengan :// jadi kita dapat memotong tambahkan **cut -d&#8221;/&#8221; -f3**

jadi **grep &#8220;href=&#8221; ghost.html | cut -d&#8221;/&#8221; -f3**

<div id="attachment_196" style="width: 675px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.me/images/2014/07/grep-domain.png"><img class="size-full wp-image-196" src="http://abdilahrf.me/images/2014/07/grep-domain.png" alt="grep domain" width="665" height="450" /></a>
  
  <p class="wp-caption-text">
    grep domain
  </p>
</div>

berhubung kita hanya ingin mencari domain dan subdomain dari website **ghost.org** jadi saya tambahkan grep &#8220;ghost.org&#8221; jadi

**grep &#8220;href=&#8221; ghost.html | cut -d&#8221;/&#8221; -f3 | grep &#8220;ghost.org&#8221;
  
** 

<div id="attachment_197" style="width: 674px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.me/images/2014/07/grep-ghost.org_.png"><img class="size-full wp-image-197" src="http://abdilahrf.me/images/2014/07/grep-ghost.org_.png" alt="grep ghost.org" width="664" height="506" /></a>
  
  <p class="wp-caption-text">
    grep ghost.org
  </p>
</div>

keliatannya hasil yang kita dapatkan hampir mendekati keinginan tapi masih banyak domain yang muncul lebih dari satu kali dan itu memungkinkan
  
kita untuk menggunakan fungsi **sort -u** untuk menghapus domain yang berulang

**grep &#8220;href=&#8221; ghost.html | cut -d&#8221;/&#8221; -f3 | grep &#8220;ghost.org&#8221;** **| sort -u**

<div id="attachment_198" style="width: 666px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.me/images/2014/07/domain-estrak.png"><img class="size-full wp-image-198" src="http://abdilahrf.me/images/2014/07/domain-estrak.png" alt="domain estrak" width="656" height="141" /></a>
  
  <p class="wp-caption-text">
    domain estrak
  </p>
</div>

dan masukan hasil ke domain.txt dengan

**grep &#8220;href=&#8221; ghost.html | cut -d&#8221;/&#8221; -f3 | grep &#8220;ghost.org&#8221;** **| sort -u** **> domain.txt
  
cat domain.txt**

<div id="attachment_199" style="width: 673px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.me/images/2014/07/to-file-txt.png"><img class="size-full wp-image-199" src="http://abdilahrf.me/images/2014/07/to-file-txt.png" alt="to file txt" width="663" height="158" /></a>
  
  <p class="wp-caption-text">
    to file txt
  </p>
</div>

setelah berhasil mengestrak domain dari file html selanjutnya kita akan gunakan list dari domain.txt untuk mendapatkan ip server dengan menggunakan script bash sederhana

** **

**http://www.offensive-security.com/**