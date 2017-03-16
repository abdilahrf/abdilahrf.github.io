---
id: 628
title: Review Deap Tree
date: 2015-06-15T22:29:25+00:00
author: abdilahrf
layout: post
guid: https://www.hasnydes.us/?p=628
permalink: /2015/06/review-deap-tree/
category: Data Structure
tags: [Kuliah]
---
Review Deap Tree , Deap Tree bisa di sebut gabungan dari max / min heap tree karena di sub-tree sebelah kiri adalah min heap  dan di sebelah kanan sub-tree adalah max heap rootnya kosong

untuk insert di dalam deap tree ini tidak memerlukan check lebih kecil atau lebih besar dari root karena memasukan input nya beurutan<!--more-->

**Rule :**

  * Root adalah blank
  * Left sub-tree adalah min heap
  * Right sub-tree adalah max heap

&nbsp;

[<img class="aligncenter size-full wp-image-629" src="http://abdilahrf.github.io/images/2015/06/insertL12.png" alt="insertL" width="420" height="332" />](http://abdilahrf.github.io/images/2015/06/insertL12.png)

Nilai paling kecil akan berada di root dari min heap , nilai paling besar akan berada di root dari max heap

Dari deap tree ini ada special , karena dia memiliki relasi terhadap node yang memiliki tempat yang sama , contohnya nomor (4,6) dan (5,7) dia memiliki tempat yang sama

Jika ada yang tidak memiliki parner di tempat yang sama contohnya (10,11) maka dia berelasi dengan relasi parent nya (8)

&nbsp;

**Insertion :**

  * Insert new node secara berurutan
  * Check node dengan relasinya (min-partner / max-partner) 
      * jika lokasinya di min-heap(left sub-tree) tapi value nya lebih besar dari partner yang berada di max-heap maka swap valuenya
      * berlaku sebaliknya
  * Lakukan min-upheap , cek ke atas seperti dengan aturan min heap
  * Lakukan max-upheap , cek ke atas seperti dengan aturan max heap

Contoh insertion

[<img class="aligncenter size-full wp-image-630" src="http://abdilahrf.github.io/images/2015/06/insertL13.png" alt="insertL" width="485" height="359" />](http://abdilahrf.github.io/images/2015/06/insertL13.png)

Karena 10 tidak memiliki partner yang sama lokasinya , maka partnernya adalah partner dari parent nya yaitu (7)  , karena 50 berada di min-heap dan 50 > dari 40 , maka swap value nya

Kemudian Kita lakukan max-upheap

[<img class="aligncenter size-full wp-image-632" src="http://abdilahrf.github.io/images/2015/06/insertL15.png" alt="insertL" width="502" height="371" />](http://abdilahrf.github.io/images/2015/06/insertL15.png)

Di cek ke atas karena 50 > 45 , sesuai dengan aturan max-heap maka swap 45 dengan 50

&nbsp;

**Deletion : **

Contoh Delete Min :

delete min berarti delete root yang ada di min-heap(5), dan ambil value terakhir yang masuk ke min-heap (25)

&nbsp;

[<img class="aligncenter size-full wp-image-633" src="http://abdilahrf.github.io/images/2015/06/insertL16.png" alt="insertL" width="647" height="247" />](http://abdilahrf.github.io/images/2015/06/insertL16.png)

&nbsp;

[<img class="aligncenter size-full wp-image-634" src="http://abdilahrf.github.io/images/2015/06/insertL17.png" alt="insertL" width="645" height="243" />](http://abdilahrf.github.io/images/2015/06/insertL17.png)

[<img class="aligncenter size-full wp-image-635" src="http://abdilahrf.github.io/images/2015/06/insertL18.png" alt="insertL" width="644" height="252" />](http://abdilahrf.github.io/images/2015/06/insertL18.png)

[<img class="aligncenter size-full wp-image-636" src="http://abdilahrf.github.io/images/2015/06/insertL19.png" alt="insertL" width="366" height="292" />](http://abdilahrf.github.io/images/2015/06/insertL19.png)

  * Masukan 25 ke variabel temp = 25 , cek temp dengan node(4) dan node(5) ,
  * karena 10 < dari 25 maka swap 25 dengan 10
  * karena 15 < dari 25 maka swap 25 dengan 15
  * cek dengan max-partner sebelum insert ke node(9) , karena 25 < 45 maka tidak perlu swap dan input 25 ke node(9)
  * lakukan min-upheap dari node(9) , karena 25 > 15 selesai

&nbsp;

&nbsp;