---
layout: post
title: "Gemastik CTF 2016 : RSA Factorization - 100"
description: "Writeup Gemastik 2016 Online Capture The Flag Qualification"
headline: 
modified: 2016-08-15
date: 2016-08-15
category: CTF
tags: [gemastik,Crypto]
imagefeature: /images/gemastik.png
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

![RSA Factorization](/images/rsa-factorization.png)


### Solusi :

Extract informasi dari key.pub dengan `openssl rsa -noout -text -inform PEM -in key.pub -pubin`

![RSA Factorization Info](/images/rsa-factorization2.png)

Public key nya mengandung Modulus N dan Exponent e. dimana N bisa di faktorisasi  `N = p x q`
Karena modulusnya berbentuk hexadecimal kita convert ke decimal untuk di faktorisasi.

`python -c "print int('5da30f696fa730d8d1df60b4d52b46e2bba689b56a0d0a8486a83568c55bc2fc150128342e63f5e537836ccbcfaf5cafed776dc0a3d3b8faaafffb770c316fb09fc44626da305ea0e49a3008dfbf7cb4f3c5ef',16)"`

Output nya :

`27997833911221327870829467638722601621070446786955428537560009929326128400107609345671052955360856061822351910951365788637105954482006576775098580557613579098734950144178863178946295187237869221823983`


Kemudian kami menggunakan 
[FactorDB 27997833...3983](http://www.factordb.com/index.php?query=27997833911221327870829467638722601621070446786955428537560009929326128400107609345671052955360856061822351910951365788637105954482006576775098580557613579098734950144178863178946295187237869221823983)
untuk mengecek bilangan prima pada angka tersebut dan hasilnya

2799783391...83<200> = 3532461934...49<100> * 7925869954...67<100>

dengan menggunakan p dan q kita bisa membuat private key, kita menggunakan [RSA Tool](https://github.com/ius/rsatool) 
untuk menggenerate private key `python rsatool.py -p primeP -q primeQ -o outputFile`

`python rsatool.py -p 3532461934402770121272604978198464368671197400197625023649303468776121253679423200058547956528088349 -q 7925869954478333033347085841480059687737975857364219960734330341455767872818152135381409304740185467 -o priv.key`

setelah private key nya terbuat maka kita bisa decrypt dengan 

`openssl rsautl -in encrypted.enc -out /dev/tty -inkey priv.key -decrypt`

Maka didapatkan flag
`GEMASTIK{no_need_for_quantum_computer_r8?}`

