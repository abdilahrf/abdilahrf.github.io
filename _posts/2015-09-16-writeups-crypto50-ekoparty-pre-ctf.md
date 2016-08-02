---
id: 1841
title: Writeups Crypto50 EkoParty Pre CTF
date: 2015-09-16T12:05:17+00:00
author: abdilahrf
layout: post
guid: https://www.hasnydes.us/?p=1841
permalink: /2015/09/writeups-crypto50-ekoparty-pre-ctf/
factory_shortcodes_assets:
  - 'a:0:{}'
categories:
  - ekoctf
---
> # Classic crypto
> 
> ##### (crypto50, solved by 149)
> 
> **Description:** Your mission is to get the hidden message!
> 
> **Attachment:** [crypto50.zip](https://ctf.ekoparty.org/static/files/crypto50.zip)

&nbsp;

Kita mendapatkan file message.mp3 , kemudian coba kita lihat strings nya

> Kx{ $5x(
  
> SIG~K
  
> #Qf)M
  
> $!~4
  
> N bX
  
> &IWV
  
> jph;
  
> 3{xs
  
> &#8230;
  
> &#8230;
  
> &#8230;
  
> UUUUUUUUUUUUUUUUUUULAME3.99.5UUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU
  
> UUUUUUUUUUUUUUUUUUUULAME3.99.5UUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU
  
> UUUUUUUUUUUUUUUUUUULAME3.99.5UUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU

hasil audacity

[<img class="aligncenter size-large wp-image-1842" src="https://www.hasnydes.us/wp-content/uploads/2015/09/audaciry-1024x576.png" alt="audaciry" width="1024" height="576" />](https://www.hasnydes.us/wp-content/uploads/2015/09/audaciry.png)

&nbsp;

Di situ banyak tulisan LAME3.99-5 itu mungkin bisa jadi clue nya atau bisa juga bukan , setelah download softwarenya dan coba2 ternyata nihil :3

kita coba menggunakan morsecode reader di android https://play.google.com/store/apps/details?id=org.jfedor.morsecode&hl=en , tapi ternyata yang di dapat dari morse code reader kata acak , bermain dengan audio sekitar 2 jam , setelah memutar ulang dengan kecepatan yang di rubah kemudian baru sadar kalau kata yang keluar di morsecode reader ternyata berulang .. dengan kecepatan yang berbeda , berarti kemungkinan besar itu adalah plaintext yang dikirim  &#8220;xxxxxxxxxxxx&#8221;

kami mencurigai itu adalah caesar cipher , kemudian kami coba decode dengan caesar tool , dengan option &#8220;test all possible key&#8221; http://www.dcode.fr/caesar-cipher kemudian kami menemukan flag di key = 6

**Flag :** EKO{morsecodereader}