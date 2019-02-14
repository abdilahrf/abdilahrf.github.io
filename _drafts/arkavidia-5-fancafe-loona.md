---
comments: true
title: 'Arkavidia 5: Fancafe Loona'
date: 2019-02-13 17:00:00 +0000
category:
- Ctf
tags:
- Arkavidia
private: false

---
### Fancafe Loona - Web

Diberikan website menggunakan `go` yang mempunyai fitur untuk mencari post dan melihat detail dari post, dengan route sebagai berikut.

![](/uploads/Capture.PNG)Kita juga diberikan akses terhadap source codenya yang bisa digunakan untuk memudahkan kita menemukan entrypoint eksploitasi pada aplikasi tersebut, dan ditemukan aplikasi sudah mengimplementasikan proteksi terhadap serangan `SQL Injection` 

![](/uploads/Capture-1.PNG)

Namun sayang nya filter tersebut belum cukup untuk mengamankan dari serangan `SQL Injection` karena dapat di bypass dengan memanfaatkan backslash `\` .