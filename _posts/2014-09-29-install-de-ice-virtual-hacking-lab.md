---
id: 246
title: Install De-ice for Virtual Hacking Lab
date: 2014-09-29T03:30:59+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=246
permalink: /2014/09/install-de-ice-virtual-hacking-lab/
categories: Pentest
tags: [Others]
---
Install De-ice for Virtual Hacking Lab , apa itu De-Ice ? [hackingdojo.com](http://hackingdojo.com)

singkatnya de-ice adalah vulnerability yang sudah di desain untuk bahan pembelajaran seperti pwnOs , kioptrix dan lain-lain . nah de-ice ini ada 3 level kalian bissa download iso nya di sini <sourceforge.net/projects/virtualhacking/files/os/de-ice/>

apa yang di butuhkan untuk install de-ice kalau ane menggunakan linux backbox jadi install vmware workstation linux x86_64 buat 64 bit

<https://www.vmware.com/go/downloadworkstation> untuk download vmware kalian harus buat akun terlebih dahulu,pilih paket installasi yang sesuai dengan komputer nya . untuk windows install seperti biasa , untuk linux gunakan command

> [<img class="aligncenter size-full wp-image-247" src="http://abdilahrf.me/images/2014/09/a.png" alt="a" width="655" height="70" />](http://abdilahrf.me/images/2014/09/a.png)

nanti munul GUI dan install vmware seperti biasa , setelah selesai install vmware buka vmware dan konfigurasi jaringannya

pilih Edit > virtual network

+ Add network > vmnet 2 > Host Only > Subnet 192.168.1.2

DHCP = Yes

[<img class="aligncenter size-full wp-image-248" src="http://abdilahrf.me/images/2014/09/dw.png" alt="dw" width="628" height="606" />](http://abdilahrf.me/images/2014/09/dw.png)

setelah selesai configurasi network , kita install de-ice pilih

  * File>New virtual Machine
  * Typical (Recomended) > Next
  * Use Iso Image > de-ice.iso > Next
  * Linux > Other Linux 2.6.x Kernel > Netx
  * Name and location default > netx
  * Split > netx
  * Custumize hardware > Network Adapter > Custom Specified > vmnet2
  * Finish

run your vmware machine

[<img class="aligncenter wp-image-249 size-full" src="http://abdilahrf.me/images/2014/09/wew-e1411960944846.png" alt="wew" width="700" height="508" />](http://abdilahrf.me/images/2014/09/wew.png)

masuk ke terminal dan ketik command ini

<div class="_5wd9" data-reactid=".2c.$mid=11411916815343=2be5044a172b2b6eb04.2:0">
  <div class="_5wde" data-reactid=".2c.$mid=11411916815343=2be5044a172b2b6eb04.2:0.0">
    <div class="_5wdf _5w1r" data-reactid=".2c.$mid=11411916815343=2be5044a172b2b6eb04.2:0.0.0">
      <blockquote>
        <div data-reactid=".2c.$mid=11411916815343=2be5044a172b2b6eb04.2:0.0.0.0">
          <span class="_5yl5" data-reactid=".2c.$mid=11411916815343=2be5044a172b2b6eb04.2:0.0.0.0.0"><span class="null">netdiscover -r <a class="_553k" href="http://192.168.1.0/24" target="_blank" rel="nofollow">192.168.1.0/24</a> -i vmnet2</span></span>
        </div>
      </blockquote>
    </div>
  </div>
</div>

nanti akan muncul ip dari vmware nya yang sedang jalanin machine De-Ice

mungkin dapat **192.168.1.100 ** kalau sudah bisa di akses server siap untuk di pentest

[<img class="aligncenter wp-image-250 size-full" src="http://abdilahrf.me/images/2014/09/Screenshot-09282014-100817-PM-e1411961290844.png" alt="Screenshot - 09282014 - 10:08:17 PM" width="800" height="450" />](http://abdilahrf.me/images/2014/09/Screenshot-09282014-100817-PM.png)

&nbsp;

Thanks : hackingdojo , muhammad alim , eric draven , double-dragon