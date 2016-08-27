---
layout: post
title: "Writeup Compfest CTF 2016 : Maze - 90"
description: "Writeup Compfest CTF 2016 Qualification"
headline: 
modified: 2016-08-27
category: compfest
tags: [compfest,ctf]
imagefeature: /images/compfest.png
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

There's a "maze" hidden within this website. Our informants told us that the flag in it has been deleted though. Recovering it would not be a problem for you, would it?

http://musashi.compfest.web.id:11663/

### Solusi :

Pertama sesuai dengan clue kita coba pikirkan kata2 yang di garis merah  
“therre aree twentty picctures”  huruf yang double adalah (retc). Ternyata tidak mengasilkan 
sesuatu yang bermanfaat. 
 
Pertama kami coba download semua jpg yang ada di web tersebut untuk di analisa lebih lanjut. 
 
wget -r http://musashi.compfest.web.id:11663 
 
Namun tidak mendapatkan hasil yang berarti 
 
Kemudian kami menemukan folder .git yang memberikan response 403 Forbidden. 
Kita Dump .git nya menggunakan GitTools  
 
 `bash gitdumper.sh http://musashi.compfest.web.id:11663/.git/ ~/workspace/CTF/compfest/web/mindthered/ `
 
Kemudian kita melakukan checkout di repositorynya 
 
`/CTF/compfest/web/ : $  git checkout -f `
 
Di dalam folder files ada folder maze yang mempunya banyak directory dan fake flag 
Untuk menemukan path flagnya  
 
`/CTF/compfest/web/ : $  ls -laR | grep -B 4 "FLAG" `
 
 
Ditemukan flag ada di `files/maze/2/1/9/7/FLAG!!!!.txt `
 
Kemudian kita mencoba membaca flag, namun sayang nya flag sudah di delete  
 
Hey! You found the fl4g! 
 
TOO BAD, YOUR FLAG IS DELETED 
 
Setelah itu kita balikan ke commit sebelum flag di hapus dengan mereset HEAD 
 
`git reset --hard HEAD~1`
 
 
Dan flag pun muncul. 
 
![Maze Flag](/images/mazeflag.png) 
 
`cat files/maze/2/1/9/7/FLAG\!\!\!\!.txt`
 
Flag : `CFCTF{h0w_d0_y0u_f1nd_th15_fl46}`


