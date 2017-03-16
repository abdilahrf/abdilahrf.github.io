---
layout: post
title: "Writeup Gemastik 2016 : Encrypted Picture - 75"
description: "Writeup Gemastik 2016 Online Capture The Flag Qualification"
headline: 
modified: 2016-08-15
category: Crypto
tags: [gemastik]
imagefeature: /images/gemastik.png
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

![Python Server](/images/encrypted-picture.png)


### Solusi :

Karena image encrypted.png di encrypt menggunakan script python di berikan dengan operasi XOR maka dengan menjalankan ulang
image yang terencrypt akan kembali ke aslinya 

![Python Server](/images/encrypted-picture-flag.png)

Ubah binary ke ascii dan muncul flag `GEMASTIK{there_is_no_sp00n}`