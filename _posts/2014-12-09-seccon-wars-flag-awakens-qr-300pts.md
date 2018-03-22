---
id: 401
title: 'Seccon CTF 2014: The Flag Awakens - QR 300pts'
date: 2014-12-09T09:18:08+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=401
permalink: /2014/12/seccon-wars-flag-awakens-qr-300pts/
category: CTF
tags: [SecconCTF,Programming]
---
> http://youtu.be/1pC56S17-_A
> 
> Not need Japanese text to solve this task.
> 
> If you need it? see below <img src="https://www.hasnydes.us/wp-includes/images/smilies/simple-smile.png" alt=":)" class="wp-smiley" style="height: 1em; max-height: 1em;" />
> 
> http://pastebin.com/uXByBZv5

<https://www.youtube.com/watch?v=1pC56S17-_A> Di youtube nya kalau kita teliti pas akhir video ada muncul barcodenya cuma di tutupin sama tulisan senccon , yang kita harus pikirkan bagaimana mendapatkan QR code secara utuh ? screenshoot manual per frame ? agak susah ,

jadi kita download videonya trus buka pake vlc karena kita bisa mengatur menggunakan filter &#8221; dari detik barcode muncul sampai selesai save tiap frame &#8220;_ _

setelah selesai pasti jadi beberapa image yang cukup banyak untuk mempersingkat waktu dan untuk hasil yang lebih akurat kita gunakan coding python untuk menggabungkan gambar per frame tadi bagian barcodenya

<pre><code class="language-python">import subprocess
from PIL import Image

width = 400
height = 300
result = Image.new('RGBA', (width, height))

img_list = subprocess.check_output(['ls', './']).split(b'\n')
i = 0
for img in img_list:
  if img[-4:] != b'.png':
    continue
  image = Image.open(img.decode('utf-8'))
  rgb_im = image.convert('RGB')
  for x in range(image.size[0]):
    for y in range(3):
      r, g, b = rgb_im.getpixel((x, image.size[1] - 1))
      result.putpixel((x, i + y),(r, g, b, 255))
    i += 3
result.save('result.png')</code></pre>

jalankan di direktori yang sama dengan pecahan gambar , kemudian hasilnya seperti ini

<a href="https://github.com/ctfs/write-ups/blob/master/seccon-ctf-2014/seccon-wars-the-flag-awakens/result.png" target="_blank"><img src="https://github.com/ctfs/write-ups/raw/master/seccon-ctf-2014/seccon-wars-the-flag-awakens/result.png" alt="" /></a>

QR scanner tidak dapat membaca dengan warna kuning dan hitam , maka beri sedikit polesan agar jadi warna hitam putih

<a href="https://github.com/ctfs/write-ups/blob/master/seccon-ctf-2014/seccon-wars-the-flag-awakens/result-modified.png" target="_blank"><img src="https://github.com/ctfs/write-ups/raw/master/seccon-ctf-2014/seccon-wars-the-flag-awakens/result-modified.png" alt="" /></a>

The flag is `SECCON{M4Y 7H3 F0RC3 83 W17H U}`.