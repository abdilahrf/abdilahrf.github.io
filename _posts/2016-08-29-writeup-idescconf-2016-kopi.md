---
layout: post
title: "Writeup Idsecconf CTF 2016 : Kopi - 50"
description: "Writeup Idsecconf CTF 2016 Online"
headline: 
modified: 2016-08-29
category: idsecconf
tags: [idsecconf,ctf,web]
imagefeature: /images/idsecconf.jpg
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

`http://128.199.96.39/`

### Solusi :

Pertama kita decompile file .class dengan java decompiler. bisa online atau offline (jd-gui).

Setelah kita dapat sourcenya kita analisa fungsi checkPassword ternyata file binary ini meminta
inputan deretan Fibbonaci dengan pemisah '-' 

`final String[] split = s.split("-");`

dan di fungsi checkPassword ada beberapa pengecekan deretan fibbonaci jadi jika yang kita
input bukan deretan fibbonaci maka akan mengasilkan `return False`.

di ketahui juga tiap buffer dikurangi dengan index fibbonaci yang kita inputkan
`buff[w] -= stack2.pop();`

contoh nya 
`ex : 103-1 = 102(f), 109-1 = 108(l)`

```python
import subprocess
import os
def fib_to(n):
     fibs = [1, 1]
     for i in range(2, n+1):
         fibs.append(fibs[-1] + fibs[-2])
     return fibs

for x in range(1,20):
     fibs = fib_to(x)
     separator = '-'.join(str(fib) for fib in fibs)
     act = os.system('java Kopi {}'.format(separator))
```

![Kopi Flag](/images/kopi_flag.png)

Flag : `flag{ILikeJavainCTF}`