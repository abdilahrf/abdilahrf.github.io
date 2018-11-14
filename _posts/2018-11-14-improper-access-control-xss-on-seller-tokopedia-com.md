---
title: Improper Access Control & XSS on seller.tokopedia.com
date: 2018-11-14 08:38
comments: true
tags:
- tokopedia
- xss
- access control
category:
- BugBounty
private: false

---
**Vulnerable Endpoint :** `http://api-id.codemi.co.id/api/v1/post/update/<COMMENTID>`

Kelemahan yang ditemukan adalah saya menemukan cara untuk membuat post pada seller.tokopedia.com yang seharusnya hanya bisa dilakukan oleh member yang memiliki role "administrator" atau "moderator" namun saya dengan akun role "learner" dapat membuat post layaknya admin maupun moderator dan juga dapat meracik XSS pada postingan tersebut.

Prosesnya untuk reproduce bug nya adalah :

* Buat komentar disalah satu post (ambil idnya)

  ![](/uploads/gambar1.png)
* Gunakan endpoint update post `http://api-id.codemi.co.id/api/v1/post/update/<commentid>` ( Seharusnya id yang dapat diterima hanya postid namun disini kita dapat mengupgrade object kita yang sebelum nya adalah komentar menjadi sebuah post dan tidak ada validasi role di endpoint tersebut )

  ![](/uploads/gambar2.png)
* Done

  ![](/uploads/gambar3.png)

  ![](/uploads/gambar4.png)

Namun sayang nya bug tersebut out of scope karena letak kelemahan ada di applikasi pihak ketiga yaitu codemi, dan tidak berhak atas reward :( tapi reward bukan akhir dari bug bounty.

![](/uploads/gambar5.PNG)

Akhir kata semoga diantara kita tidak takut untuk sharing karena sharing tidak akan membuat kita miskin melainkan sebaliknya akan membuat kita lebih dalam segala hal, semoga hari kita menyenangkan terutama yang membaca tulisan ini :D