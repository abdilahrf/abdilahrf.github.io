---
id: 593
title: Pengenalan Binary Search Tree
date: 2015-06-03T19:43:47+00:00
author: abdilahrf
layout: post
guid: https://www.hasnydes.us/?p=593
permalink: /2015/06/pengenalan-binary-search-tree/
factory_shortcodes_assets:
  - 'a:0:{}'
category: Algorithm
tags: [Algorithm]
---
**Pengenalan Binary Search Tree** &#8211; Binary Search Tree bisa di singkat (BST) adalah sebuat binary tree , biasanya memiliki ciri-ciri seperti berikut :

  * Setiap node mempunyai value dan tidak ada value yang double
  * value yang ada di kiri tree lebih kecil dari rootnya
  * value yang ada di kanan tree lebih besar dari rootnya
  * kiri dan kanan tree bisa menjadi root lagi atau bisa mempunya child jadi BST ini memiliki sifat ( rekrusif )

&nbsp;

<div id="attachment_594" style="width: 518px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.github.io/images/2015/06/Picture1.png"><img class="size-full wp-image-594" src="http://abdilahrf.github.io/images/2015/06/Picture1.png" alt="Binary Search Tree" width="508" height="452" /></a>
  
  <p class="wp-caption-text">
    Binary Search Tree
  </p>
</div>

&nbsp;

<!--more-->dalam contoh di atas setiap node memiliki value yang berbeda , dan sesuai dengan ciri-ciri yang di sebutkan di atas

Binary Search Tree memiliki 3 oprasi dasar ,  hampir mirip dengan array keliatannya jadi ada 3 yaitu :

  1. Find(x)     : find value x didalam BST ( Search )
  2. Insert(x)   : memasukan value baru x ke BST ( Push )
  3. Remove(x)  : menghapus key x dari BST ( Delete )

&nbsp;

### Oprasi: Search BST

dengan adanya ciri&#8221; atau syarat di dalam BST , maka untuk finding/searching didalam BST menjadi lebih mudah.

Bayangkan kita akan mencari value X.

  * Memulai Pencarian Dari Root
  * Jika Root adalah value yang kita cari , maka berhenti
  * Jika x lebih kecil dari root maka cari kedalam rekrusif tree sebelah kiri
  * Jika x lebih besar dari root maka cari kedalam rekrusif tree sebelah kanan

### Operasi: Insertion BST

Memasukan value (data) baru kedalam BST dengan rekrusif

Bayangkan kita menginsert value x :

  * Dimulai dari root
  * jika x lebih kecil dari node value(key) kemudian cek dengan sub-tree sebelah kiri lakukan pengecekan secara berulang ( rekrusif )
  * jika x lebih besar dari node value(key) kemudian cek dengan sub-tree sebelah kanan lakukan pengecekan secara berulang ( rekrusif )
  * Ulangi sampai menemukan node yang kosong untuk memasukan value X ( X akan selalu berada di paling bawah biasa di sebut **Leaf** atau daun )

#### Contoh Insertion :

Kita ingin menambahkan value **35 **kedalam BST yang sudah ada

[<img class="aligncenter size-full wp-image-595" src="http://abdilahrf.github.io/images/2015/06/Picture2.png" alt="Insertion Binary Search Tree" width="508" height="535" />](http://abdilahrf.github.io/images/2015/06/Picture2.png)

yang berwarna hijau adalah root , setiap menginsert , mencari , atau delete selalu di mulai dari root.

dan new node berwarna orange yang memiliki value 35, kemudian kita cek dengan root(30), maka 30 .. 35 ?  karena 30 < 35 maka , node lebih besar dari root kemudian kita arahkan ke sub-tree yang berada di kanan dan kemudian cek ulang kembali

[<img class="aligncenter size-full wp-image-596" src="http://abdilahrf.github.io/images/2015/06/Picture3.png" alt="Insertion Example BST" width="508" height="535" />](http://abdilahrf.github.io/images/2015/06/Picture3.png)

&nbsp;

Sekarang kita cek 35 dengan 37 , maka 35 < 37 jadi kita arahkan ke bagian kiri dan kita cek kembali , karena di bagian kiri sudah ada value yaitu 32

[<img class="aligncenter size-full wp-image-597" src="http://abdilahrf.github.io/images/2015/06/Picture4.png" alt="Insertion BST" width="507" height="534" />](http://abdilahrf.github.io/images/2015/06/Picture4.png)

kemudian kita cek 32 dengan 35 , ternyata 35 > 32 jadi kita masukan ke kanan , dan ternyata kita sudah menemukan tempat kosong untuk mengisi new node(35) jadi kita masukan 35 di sebelah kanan dari node(32)

[<img class="aligncenter size-full wp-image-598" src="http://abdilahrf.github.io/images/2015/06/Picture5.png" alt="Insertion BST" width="509" height="444" />](http://abdilahrf.github.io/images/2015/06/Picture5.png)

### Operasi: Delete ( Remove )

akan ada 3 case yang ditemukan ketika ingin menghapus yang perlu diperhatikan :

  * Jika value yang ingin dihapus adalah Leaf(Daun) atau paling bawah , langsung delete
  * Jika value yang akan dihapus mempunyai satu anak, hapus nodenya dan gabungkan anaknya ke parent value yang dihapus
  * jika value yang akan di hapus adalah node yang memiliki 2 anak , maka ada 2 cara , kita bisa cari dari left sub-tree anak kanan paling terakhir(leaf) (kiri,kanan~~) atau dengan cari dari right sub-tree anak kiri paling terakhir(leaf) (kanan,kiri~~)

#### Contoh Delete

Delete value 21

[<img class="aligncenter size-full wp-image-599" src="http://abdilahrf.github.io/images/2015/06/Picture6.png" alt="Delete BST" width="507" height="436" />](http://abdilahrf.github.io/images/2015/06/Picture6.png)

21 adalah **Leaf **atau node paling bawah jadi tinggal delete

&nbsp;

<div id="attachment_600" style="width: 517px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.github.io/images/2015/06/Picture7.png"><img class="size-full wp-image-600" src="http://abdilahrf.github.io/images/2015/06/Picture7.png" alt="Delete BST" width="507" height="436" /></a>
  
  <p class="wp-caption-text">
    Delete BST
  </p>
</div>

&nbsp;

#### Contoh Delete 2

[<img class="aligncenter size-full wp-image-601" src="http://abdilahrf.github.io/images/2015/06/Picture8.png" alt="Picture8" width="507" height="442" />](http://abdilahrf.github.io/images/2015/06/Picture8.png)

kita ingin delete node(37) memiliki 2 anak , jadi kita bisa ambil  (35 atau 42 ) untuk menggantikan value(37)

Kiri , anak paling kanan atau kanan anak paling kiri

[<img class="aligncenter size-full wp-image-602" src="http://abdilahrf.github.io/images/2015/06/Picture9.png" alt="Picture9" width="507" height="442" />](http://abdilahrf.github.io/images/2015/06/Picture9.png)

#### Contoh 3 Delete

[<img class="aligncenter size-full wp-image-603" src="http://abdilahrf.github.io/images/2015/06/Picture10.png" alt="Picture10" width="509" height="562" />](http://abdilahrf.github.io/images/2015/06/Picture10.png)

Delete value(30) , value 30 adalah root dari BST , memiliki 2 anak sama seperti case sebelumnnya _&#8220;Kiri , anak paling kanan atau kanan anak paling kiri&#8221;._

jadi kita bisa ambil ( 26 atau 31 ) untuk menggantikan root(30)

[<img class="aligncenter size-full wp-image-604" src="http://abdilahrf.github.io/images/2015/06/Picture11.png" alt="Picture11" width="509" height="437" />](http://abdilahrf.github.io/images/2015/06/Picture11.png)

&nbsp;

&nbsp;