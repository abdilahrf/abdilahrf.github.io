---
layout: post
title: "AkiCTF Writeups"
description: "Writeup AkiCTF"
headline: 
modified: 2016-11-09
date: 2017-01-30
imagefeature: 
mathjax: 
chart: 
category: CTF
tags: [AkiCTF]
comments: true
featured: false
---

## Writeup AkiCTF

ini sebenernya ctf yang udah lama kayaknya ini mulai pas tahun 2013, tapi 
challenge nya menurut gw masih legit buat di cobain link nya di sini [AkiCTF](http://ctf.katsudon.org/)


### Game #1 - 70 Points

Deskripsi soal nya gak ada sama sekali, kita cuma di kasih link [q6.ctf](http://q6.ctf.katsudon.org)

Dari website nya kita bisa liat di situ ada game RPS(rock, papper,scissor) dan di bagian bawahnya ada 
rangking yang kalau sepertinya kalau duit kita cukup untuk beli ranking nya flagnya bakal muncul

view pagesource nya kita bisa liat di situ ada kode javascript berikut :

``` javascript 

<script>
    $(function() {
        var d = $("#h"),     // id h 
            c = $("#money"), // duit kita diambil dari sini
            b = $("#message"); // id message 
        
        $("#jankenForm").submit(function() { // kalau form RPS nya kita submit
            0 < c.text() && $.post("/janken", { // kirim post 
                h: d.val(), // parameter h di ambil dari variable d.val()
                money: c.text() // parameter money di ambil dari c.text
            }).done(function(a) { 
                a.error ? b.text("error") : (d.val(a.h), c.text(a.money), b.text(a.message))
            }).fail(function() {
                b.text("Oops... Try again later.")
            });
            return !1
        });
        
        $("#registerForm").submit(function() {
            $.post("/register", {
                h: d.val(),
                money: c.text()
            }).done(function(a) {
                a.error ? b.text("error") : b.text(a.message)
            }).fail(function() {
                b.text("Oops... Try again later.")
            });
            return !1
        })
    });
</script>
```

kalau kita liat javascriptnya dia kan ambil money langsung dari #money tapi kalau kita ganti
misalnya inspect element jadi 99999999 terus kita coba register bakal output nya jadi error
kenapa? karena pas di post ada 2 parameter yang di lempar selain parameter money, yaitu parameter h 

isi variable h adalah md5 `d3d9446802a44259755d38e6d163e820 = 10` jadi yang di kirim di parameter h adalah
hash md5 dari money yang nanti bakal di validasi lagi di server nya jadi untuk ngecoh validasinya
kita modifikasi #money sama value dari h

jadi kita bisa langsung kirim ke register dan atur duitnya sesuka kita : 
`curl 'http://q6.ctf.katsudon.org/register' -H 'Content-Type: application/x-www-form-urlencoded; charset=UTF-8' -H 'X-Requested-With: XMLHttpRequest' --data 'h=c8c605999f3d8352d7bb792cf3fdb25b&money=999999999' --compressed`

flag bakal muncul di response nya 
`{"message":"Congrats! The flag is "XXXXXXXXX"."}`



### From login form - 120 Points

To be continue,