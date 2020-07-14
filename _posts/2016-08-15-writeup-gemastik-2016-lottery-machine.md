---
layout: post
title: "Gemastik CTF 2016 : Lottery Machine - 125"
description: "Writeup Gemastik 2016 Online Capture The Flag Qualification"
headline: 
modified: 2016-08-15
date: 2016-08-15
category: ctf
tags: [gemastik,Pwnable]
imagefeature: /images/gemastik.png
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

![Lottery Machine](/images/lottery-machine.png)


### Solusi :

Didalam codenya deklarasi array untuk guessed_slot ada tepat dibawah lottery_slot, begitu juga pada saat pengesettan 
memory dengan memset yang menandakan bahwa lokasi memory untuk 
guessed_slot tepat berada diatas lottery_slot pada urutan memory stack...

![Code1](/images/code1.png)

Untuk dapat mengakses lotterynya kita input nilai negative Lalu sisanya kita berikan karakter
bebas (symbol/huruf) untuk menghentikan programnya Dan akhirnya flagnya pun ke print

![Lottery Machine Flag](/images/lottery-machine-flag.png)

