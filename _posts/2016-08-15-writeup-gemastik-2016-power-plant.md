---
layout: post
title: "Writeup Gemastik 2016 : Power Plant - 125"
description: "Writeup Gemastik 2016 Online Capture The Flag Qualification"
headline: 
modified: 2016-08-15
date: 2016-08-15
category: Reverse
tags: [gemastik]
imagefeature: /images/gemastik.png
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

![Power plant](/images/power-plant.png)


### Solusi :

Ketika kita mencoba melakukan koneksi ke service kita di tampilkan dengan input yang
meminta untuk memasukan Access Code.Kemudian kita mencoba melihat source code nya
untuk mengetahui bagaimana Access Code itu di validasi.

![Power Plant Redacted](/images/power-plant-1.png)

Sayangnya di source code ternyata tidak memberikan full source. Jadi memaksa kita untuk
melakukan reverse engineering.

![Power Plant IDA](/images/power-plant-ida.png)

Kami menggunakan IDA Pro untuk menganalisa fungsi yang di redacted pada source code C nya. Di pseudocode terlihat ada variable 
&unk_8048940 yang membandingkan dengan v5[21]. ada urutan hex yang di simpan di &unk_8048940

![Power Plant Hex](/images/power-plant-hex.png)

“40 4A 49 55 4A 4F 4B 3A 38 49 3A 36 38 3E 40 3E 36 42 3C 41 3E” untuk mendapatkan plaintext dari access code nya 
kita melakukan aritmatika kebalikan pada pseudocode adalah substract dari bitwise.

```c
   v3 = 0;
    for ( i = 0; i <= 20; ++i 
)
    {
      if ( v5[i] == ~i + a1[i] 
)
        ++v3;
    }
```

Menggunakan python 

![Power Plant Convert](/images/power-plant-convert.png)

Di dapatkan access code :

![Power Plant Access Code](/images/access-code.png)

Dan FLAG `GEMASTIK{_______________all_ur_c0de_belong_2_us}`

![Power Plant Flag](/images/power-plant-flag.png)

