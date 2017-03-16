---
id: 274
title: EL Injection
date: 2014-10-26T13:02:16+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=274
permalink: /2014/10/el-injection/
categories: Web Exploitation
tags: [Others]
---
<span style="color: #000000;"><strong>El injection(Expression Language Injection)</strong></span>

&nbsp;

Apa itu Expression language : expression language adalah bahasa pemrograman bertujuan khusus untuk dengan mudah mengakses data aplikasi yang tersimpan di Java Beans components,system objects,managing application,devices and services. Mereka digunakan untuk embedding ekspresi seperti aritmatika dan ekspresi logis dalam halaman web.
  
Admin menggunakannya untuk melakukan tugas-tugas seperti membaca data, menulis data, dll

Siapa yang menemukan El Injection ? Sir Stefano Di Paola, adalah orang yang menemukan kerentanan yang menarik ini pada tahun 2011.
  
Jadi, sekarang Anda mungkin sudah menebak apa yang dapat dilakukan dengan injeksi EL. Aplikasi ditargetkan memutuskan lingkup kerentanan.
  
Memanfaatkan kerentanan yang dapat memberikan hasil dengan XSS, RCE dan juga dapat mengambil data dari database! Kerentanan terjadi ketika sebuah aplikasi web adalah menjalankan versi lama dari clearspace yang sekarang dikenal sebagai Jive Software.
  
Clearspace adalah alat manajemen pengetahuan dan terintegrasi dengan Spring Framework.
  
Clearspace rentan terhadap bug ini karena pola EL digunakan dalam JSP Tag.

<!--more-->

Bagaimana cara penyerangannya ? contohnya seperti ini

kita kirim dengan get unauth dan kita isi proses aritmatika 100 x 3 **<a class="bbc_link" href="https://target-site/login!input.jspa?unauth" target="_blank">https://target-site/login!input.jspa?unauth=</a>${100*3}**. dan nanti pada source nya akan muncul hasil dari 100*3

[<img class="aligncenter size-full wp-image-275" src="http://abdilahrf.me/images/2014/10/s652sx.png" alt="s652sx" width="638" height="222" />](http://abdilahrf.me/images/2014/10/s652sx.png)

&nbsp;

dan kita coba dengan query yang lain  **<a class="bbc_link" href="https://target-site/login!input.jspa?unauth" target="_blank">https://target-site/login!input.jspa?unauth=</a>${99999+1}**

[<img class="aligncenter size-full wp-image-276" src="http://abdilahrf.me/images/2014/10/oye.png" alt="oye" width="637" height="203" />](http://abdilahrf.me/images/2014/10/oye.png)

dari hasil seperti ini mungkin bisa di explore sampai bisa maitening access atau sebagainya

bagaimana cara patching ?

yang harus di lakukan adalah **update**

Official Paper: <a class="bbc_link" href="http://www.mindedsecurity.com/fileshare/ExpressionLanguageInjection.pdf" target="_blank">http://www.mindedsecurity.com/fileshare/ExpressionLanguageInjection.pdf</a>
  
Read More: <a class="bbc_link" href="https://www.owasp.org/index.php/Expression_Language_Injection" target="_blank">https://www.owasp.org/index.php/Expression_Language_Injection</a>

Sources : https://evilzone.org/tutorials/el-injection