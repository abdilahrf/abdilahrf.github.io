---
layout: post
title: 'Writeup FIT FUNFEST 2017 : EncryptedELF'
description: Writeup EncryptedELF FIT FUNFEST CTF
modified: 'Thu Mar 16 2017 07:00:00 GMT+0700 (WIB)'
category: Crypto
tags: FIT CTF
imagefeature: /images/fit-funfest.png
mathjax: false
chart: null
comments: true
featured: false
published: true
private: 'false'
---


### Soal

```sh
$ file myelf
myelf: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked,
interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32,
BuildID[sha1]=a117956f8826f8ccb19ac6be9d48808c66d1f357, not stripped
Dapatkan kembali file myelf yang telah dienkripsi menggunakan encrypt.py!
http://139.59.233.122/soal/crypto/encrypted-elf-revised/encrypt.py
http://139.59.233.122/soal/crypto/encrypted-elf-revised/encryptedelf
```

### Solusi

Goals nya Membuat fungsi untuk melakukan dekripsi ELF nya dengan memanfaatkan IV dan
KEY yang ada di append ke akhir binary dan di XOR dengan magic header ELF
binary

Kami membuat script python untuk mengambil IV dan KEY dari bawah file yang
terencrypt kemudian memisahkan chunks dengan filesize yang ada pada 8 byte
awal file encryptedelf​ dan kemudian kami melakukan recovery IV dan KEY dengan
melakukan XOR dengan magic header ELF Binary

```python
def decrypt_file(in_filename, out_filename, chunksize=64*1024):
    filesize = os.path.getsize(in_filename)
    with open(in_filename, 'rb') as infile:
        with open(out_filename, 'wb') as outfile:

            chunk = infile.read(chunksize)
            ch = chunk[8:-32]
            chn = ['\x7f', 'E', 'L', 'F', '\x02', '\x01', '\x01', '\x00', '\x00', '\x00', '\x00', '\x00', '\x00', '\x00', '\x00', '\x00']

            v = chunk[-32:-16:]
            k = chunk[-16:]

            iv = ''.join(chr(ord(chn[i]) ^ ord(v[i])) for i in range(0, 16))

            print "IV: "+ iv
            key = ''.join(chr(ord(chn[i]) ^ ord(k[i])) for i in range(0, 16))
            print "KEY: "+ key

            handler = AES.new(key, AES.MODE_CBC, iv)
            outfile.write(handler.decrypt(ch))
            
decrypt_file('encryptedelf','origelf')
```
![encryptedelf-flag.png]({{site.baseurl}}/images/encryptedelf-flag.png)

Kemudian di jalankan scriptnya untuk melakukan decrypt.

**Flag : FIT2017{this_time_is_modern_cipher}**
