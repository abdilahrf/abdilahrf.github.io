---
private: 'false'
layout: post
featured: false
published: true
comments: true
modified: 2017-08-21
date: 2017-08-21
title: Writeup final beefest 2017
tags: beefest
mathjax: false
imagefeature: 
description: Writeup final beefest 2017
category: Writeup
chart: null
---


## Logic

```
ssh 10.22.130.222:22
username : user1
pass : user1
read source code di /home/binex1/binex1.cpp 
binary ada di /home/binex1/binex1
Hint : System Variable
Hint : Use $PATH Environment
```


## POC

kita bisa manfaatin $PATH Environment untuk mengelabui program binex1.cpp yang memanggil 
`system("id")` dia bakal pake binary id yang ada di $PATH `/usr/bin/id` karena kita punya kendai terhadap
$PATH kita bisa bikin folder baru di `/tmp` misalnya `/tmp/fakebin` terus kita bikin binary `id` 
yang nge-link ke `/bin/sh` dengan command `ln -s /bin/sh /tmp/fakebin/id`

kemudian kita tinggal modifikasi $PATH nya dengan `export PATH=/tmp/fakebin` kemudian
balik ke `cd /home/binex1/` jalanin binary `./binex1` nanti `system("id")` akan manggil
`/tmp/fakebin/id` yang akan ngetrigger `/bin/sh` dan kita bisa baca flagnya dengan `cat flag.txt`

Flag: **CTFS{Abuse_the_system!!!}**




## Overflow

```
overflow
ssh 10.22.130.222:22
username : user2
pass : user2
file binary ada di /home/binex2/binex2
```

## POC

binary bisa di download menggunakan scp atau semacamnya. setelah di download binarnya
lakukan decompile menggunakan IDA Pro harusnya terlihat ada fungsi yang membandingkan apakah
secret nya udah ter-overflow dengan hex "0xc0deface"

ketika kondisinya sudah terpenuhi maka `system("cat flag.txt")` akan ter trigger.


Flag: **CTFS{w00t_the_flag_is_showing}**



## php is PHP

```
Read and pwn get the flag!

http://10.22.130.222/phpisPHP/

Hint: PHP Banyak bug :)
```

diberikan website yang meminta inputan flag :) dan source codenya.


## POC

untuk solve soal ini sebenenarnya kita hanya perlu mengirimkan request field name nya 
dalam bentuk array untuk membypass validasi dengan `strcasecmp` pada source code ada beberapa
code sebagai pengecoh ;) `strcasecmp` adalah fungsi tetangga nya dari `strcmp`

Flag: **CTFS{remember_strcasecmp_is_another_strcmp_h3h3}**


## Injectme
```
Banyak yang di blackLIST!


http://10.22.130.222/injectme/

 

Hint : username untuk login di filter dengan fungsi secure dibawah


function secure($data) {
    if (preg_match("/username|USERNAME|password|PASSWORD|union|select|UNION|SELECT|from|FROM|sleep|id|ID|SLEEP|ascii|hex|ASCII|HEX|binary|BINARY|chr|CHR| /", $data)) {
        $bl = "username|USERNAME|password|PASSWORD|union|select|UNION|SELECT|from|FROM|sleep|id|ID|SLEEP|ascii|hex|ASCII|HEX|binary|BINARY|chr|CHR";
        $bl = explode("|", $bl);
        foreach ($bl as $b) {
            if (strstr($data, $b)) {
                echo $b . " is blaklisted 
";
            }
        }
        die("You cant use input blacklisted");
    }
    return $data;
}
```

## POC

Kita bisa mendapatkan flag dengan menggunakan union based, filter yang di gunakan pada
fungsi secure masih sangat longgar dan bisa di bypass karena preg_match yang digunakan
hanya mengecek `select` dan `SELECT` kita bisa menggunakan `seLect` sebagai bypass, begitu
juga dengan keyword yang lainnya.

pada fungsi secure juga " " (spasi) di blacklist, kita bisa ganti menggunakan `/**/` sebgagai
pengganti spasi

kurang lebih payload yang work sebagai berikut `'/**/unIon/**/seLect/**/1,(seLect/**/pasSword/**/fRom/**/users)#`

Flag : **CTFS{tHi$_p4zZw0rd_h4rD_en0ugH}**


## Basic

Diberikan flag yang di encode BASE32

## POC

gunakan utility di linux `echo -n "BASE32FLAG" | base32 -d`

Flag: **CTFS{1m_4_b4s3_m4st3r}**


## Meta sih anak nakal

```
Meta suka banget ngumpetin data di gambar, bisa gak temuin flag yang dia sembunyikan ? 

Meta sih anak nakal.zip
```

## POC

`exiftool img.jpg`

Flag: **CTFS{m4t4d4t4_1tu_p3nt1ng}**


