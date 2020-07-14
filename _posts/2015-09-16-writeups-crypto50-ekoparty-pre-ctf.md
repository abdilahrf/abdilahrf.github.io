---
id: 1841
title: 'EkoParty Pre-CTF 2015: Crypto50'
date: 2015-09-16T12:05:17+00:00
author: abdilahrf
layout: post
guid: https://www.hasnydes.us/?p=1841
permalink: /2015/09/writeups-crypto50-ekoparty-pre-ctf/
categories: ctf
tags: [IceCTF,Crypto]
---
> # Classic crypto
> 
> ##### (crypto50, solved by 149)
> 
> **Description:** Your mission is to get the hidden message!
> 
> **Attachment:** [crypto50.zip](https://ctf.ekoparty.org/static/files/crypto50.zip)

&nbsp;

Kita mendapatkan file message.mp3 , kemudian coba kita lihat strings nya tidak mengandung sesuatu yang memiliki informasi


hasil audacity

[<img class="aligncenter size-large wp-image-1842" src="http://abdilahrf.github.io/images/2015/09/audaciry-1024x576.png" alt="audaciry" width="1024" height="576" />](http://abdilahrf.github.io/images/2015/09/audaciry.png)

&nbsp;

kita coba menggunakan morsecode reader di android https://play.google.com/store/apps/details?id=org.jfedor.morsecode&hl=en , tapi ternyata yang di dapat dari morse code reader kata acak , bermain dengan audio sekitar 2 jam , setelah memutar ulang dengan kecepatan yang di rubah kemudian baru sadar kalau kata yang keluar di morsecode reader ternyata berulang .. dengan kecepatan yang berbeda , berarti kemungkinan besar itu adalah plaintext yang dikirim  &#8220;xxxxxxxxxxxx&#8221;

kami mencurigai itu adalah caesar cipher , kemudian kami coba decode dengan caesar tool , dengan option &#8220;test all possible key&#8221; http://www.dcode.fr/caesar-cipher kemudian kami menemukan flag di key = 6

**Flag : EKO{morsecodereader}**