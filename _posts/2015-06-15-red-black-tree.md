---
id: 611
title: Review Red Black Tree
date: 2015-06-15T21:25:59+00:00
author: abdilahrf
layout: post
guid: https://www.hasnydes.us/?p=611
permalink: /2015/06/red-black-tree/
category: DataStructure
tags: [Kuliah]
---
Red black tree sebenarnya memiliki beberapa rule yang masih sama dengan AVL tree , seperti jika lebih kecil cek ke sub-tree sebelah kiri jika lebih besar cek ke sub-tree sebelah kanan , tapi bedanya di RBT ini menggunakan warna hitam dan merah

**Rule :**

  * Root harus selalu hitam
  * New Node adalah merah
  * External node atau null node adalah hitam
  * Jika  node merah tanpa sibling , memiliki anak merah maka lakukan rotate (single/double)  #case1
  * Jika node merah dengan sibling , memiliki anak merah maka turunkan warna dari grand parent  #case2
  * Jika RBT tidak seimbang maka rotate (single/double) , single = ( kanan-kanan atau kiri-kiri ) , double = ( kanan-kiri atau kiri-kanan) #case3

<!--more-->

**In Example : **

Insert &#8220;LABCOMPUTING&#8221; to Red Black Tree

**insert L** 

insert dengan warna merah , karena L adalah root maka ganti ke hitam

[<img class="aligncenter size-full wp-image-612" src="http://abdilahrf.github.io/images/2015/06/insertL.png" alt="insertL" width="194" height="125" />](http://abdilahrf.github.io/images/2015/06/insertL.png)

**Insert A**

karena A < L maka cek ke sub-tree sebelah kiri , kalau kosong langsung input node

[<img class="aligncenter size-full wp-image-613" src="http://abdilahrf.github.io/images/2015/06/insertL1.png" alt="insertL" width="323" height="184" />](http://abdilahrf.github.io/images/2015/06/insertL1.png)

&nbsp;

**Insert B**

  * karena B < L maka cek ke sub-tree sebelah kiri ,
  * karena B > A maka cek sub-tree sebelah kanan , kalau kosong langsun input node
  * kita mendapatkan #case1 , parent dengan warna merah memiliki anak merah maka lakukan rotate , disini double rotate

&nbsp;

&nbsp;

<div id="attachment_615" style="width: 310px" class="wp-caption aligncenter">
  <a href="https://www.hasnydes.us/2015/06/red-black-tree/doublerotate/" rel="attachment wp-att-615"><img class="wp-image-615 size-full" src="http://abdilahrf.github.io/images/2015/06/doublerotate.gif" alt="doublerotate" width="300" height="300" /></a>
  
  <p class="wp-caption-text">
    Animasi Double Rotation
  </p>
</div>

[<img class="aligncenter size-full wp-image-614" src="http://abdilahrf.github.io/images/2015/06/insertL2.png" alt="insertL" width="431" height="186" />](http://abdilahrf.github.io/images/2015/06/insertL2.png)

&nbsp;

**Insert C**

  * karena C > B maka , cek sub-tree kanan ,
  * karena C < L maka cek sub-tree kiri , kalau kosong lagsung masukan node
  * tapi kita menemukan #case2 parent merah yang memiliki sibling merah dan anak merah , jadi kita harus turunkan warna dari root ke (A,L) jadi hitam
  * kemudian warna root kembali jadi hitam (B)

**[<img class="aligncenter size-full wp-image-616" src="http://abdilahrf.github.io/images/2015/06/insertL3.png" alt="insertL" width="473" height="240" />](http://abdilahrf.github.io/images/2015/06/insertL3.png)**

**Insert O**

  * O > B , cek sub-tree kanan
  * O > L , cek sub-tree kanan , jika kosong input node

[<img class="aligncenter size-full wp-image-617" src="http://abdilahrf.github.io/images/2015/06/insertL4.png" alt="insertL" width="654" height="230" />](http://abdilahrf.github.io/images/2015/06/insertL4.png)

&nbsp;

**Insert M **

  * M > B , cek sub-tree kanan
  * M > L , cek sub-tree kanan
  * M < O , cek sub-tree kiri , kosong input node
  * Kita menemukan #case2 lagi jadi kita ambil warna dari grand parent(L) , turun ke (C,O)

[<img class="aligncenter size-full wp-image-618" src="http://abdilahrf.github.io/images/2015/06/insertL5.png" alt="insertL" width="636" height="309" />](http://abdilahrf.github.io/images/2015/06/insertL5.png)

&nbsp;

**Insert P**

  * R > B , cek sub-tree kanan
  * R > L , cek sub-tree kanan
  * R > O , cek sub-tree kanan, kosong input node

<div id="attachment_619" style="width: 591px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.github.io/images/2015/06/insertL6.png"><img class="wp-image-619 size-full" src="http://abdilahrf.github.io/images/2015/06/insertL6.png" alt="insertL" width="581" height="303" /></a>
  
  <p class="wp-caption-text">
    R di gambar seharusnya P
  </p>
</div>

**Insert U**

  * U > L , cek sub-tree kanan
  * U > O , cek sub-tree kanan
  * U > P , cek sub-tree kanan , kosong input node
  * kita menemukan #case2 kembali ganti warna (M,P) menjadi hitam , ambil dari grand parent (O) ,turunkan lagi warna dari (B) ke (A,L)
  * karena RBT tidak seimbang , dalam case ini (single rotation) , kanan-kanan , maka lakukan single rotation dari root

[<img class="aligncenter size-full wp-image-620" src="http://abdilahrf.github.io/images/2015/06/insertL7.png" alt="insertL" width="639" height="288" />](http://abdilahrf.github.io/images/2015/06/insertL7.png)

**Insert T**

  * T > L , cek sub-tree kanan
  * T > O , cek sub-tree kanan
  * T > P , cek sub-tree kanan
  * T < U , input node
  * kita bertemu dengan #case1 ( double rotation )

[<img class="aligncenter size-full wp-image-621" src="http://abdilahrf.github.io/images/2015/06/insertL8.png" alt="insertL" width="678" height="295" />](http://abdilahrf.github.io/images/2015/06/insertL8.png)

**Insert I**

  * I < L , cek sub-tree kiri
  * I > B , cek sub-tree kanan
  * I > C , cek sub-tree kanan , input node

[<img class="aligncenter size-full wp-image-622" src="http://abdilahrf.github.io/images/2015/06/insertL9.png" alt="insertL" width="655" height="257" />](http://abdilahrf.github.io/images/2015/06/insertL9.png)

**Insert N**

  * N > L , cek sub-tree kanan
  * N < O , cek sub-tree kiri
  * N > M , input node

[<img class="aligncenter size-full wp-image-623" src="http://abdilahrf.github.io/images/2015/06/insertL10.png" alt="insertL" width="810" height="306" />](http://abdilahrf.github.io/images/2015/06/insertL10.png)

**Insert G**

  * G < L , cek sub-tree kiri
  * G > B , cek sub-tree kanan
  * G > I , cek sub-tree kanan

[<img class="aligncenter size-full wp-image-624" src="http://abdilahrf.github.io/images/2015/06/insertL11.png" alt="insertL" width="856" height="384" />](http://abdilahrf.github.io/images/2015/06/insertL11.png)

&nbsp;

&nbsp;