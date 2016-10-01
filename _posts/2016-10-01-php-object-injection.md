---
layout: post
title: "PHP Object Injection Via Unserialize"
description: "PHP Object Injection Via Unserialize"
headline: 
modified: 2016-10-01
category: Web Exploitation
tags: [Web Exploitation]
imagefeature: /images/php.jpg
mathjax: 
chart: 
comments: true
featured: true
---

PHP Unserialize bisa menjadi petaka ketika kita mengambil atau menggunakan input
dari user untuk di eksekusi ke unserialize(), dalam dokumentasi fungsi [unserialize()](http://php.net/manual/en/function.unserialize.php) di php.net
kita sudah di beri warning untuk tidak menggunakan input dari user karena bisa
berdampak code execution pada saat object insantiation atau autoloading, untuk keamanan
kita bisa menggunakan [json_encode()](http://php.net/manual/en/function.json-decode.php) atau [json_decode()](http://php.net/manual/en/function.json-encode.php) sebagai standar pertukaran data.

Sebelum melanjut ke tahap melakukan exploitasi pada kasus yang vulnerable mari kita pahami
apa yang di lakukan oleh fungsi [serialize()](http://php.net/manual/en/function.serialize.php) dan [unserialize()](http://php.net/manual/en/function.unserialize.php)


## Serialize

`string serialize ( mixed $value )`

[serialize()](http://php.net/manual/en/function.serialize.php) menerima input satu parameter yaitu
value yang ingin di serialize dalam bentuk apapun kecuali [resource](http://php.net/manual/en/language.types.resource.php)-type
Return value yang di kembalikan adalah byte-stream representasi dari value.

#### Contoh

```php
<?php
$myArr = array("PHP","IS","NICE");
$serialized = serialize($myArr);
print $serialized;
?>
```
Output : `a:3{i:0;s:3:"PHP";i:1;s:2:"IS";i:2;s:4:"NICE";}`

- `a:3{`          : Array dari 3 value
- `i:0;`          : Integer, Index ke [0]
- `s:3:"PHP"`     : String, 3 karakter, Value "PHP"
- `i:1;`          : Integer, Index ke [1]
- `s:2:"IS"`      : String, 2 karakter, Value "IS"
- `i:2;`          : Integer, Index ke [2]
- `s:4:"NICE"}`   : String, 4 karakter, Value "NICE"

Kira-kira seperti itu penjabaran dari serialize nya.

## Unserialize()

`mixed unserialize ( string $str [, array $options ] )`

*[unserialize()](http://php.net/manual/en/function.unserialize.php)  takes a single serialized variable and converts it back into a PHP value.*
Jadi unserialize melakukan konversi dari serialize object kedalam kode PHP kembali.

selain menggunakan json sebagai standar pertukaran data Saran lain dari dokumentasi 
php.net adalah untuk menggunakan [hmac](http://php.net/manual/en/function.hash-hmac.php)
untuk melakukan validasi dan menjaga data tidak ada yang memodifikasi selain dari yang di tentukan server.

Note : $options parameter baru ada di changelog php 7.0.0

#### Contoh

```php
<?php
$value=‘a:1:{s:4:"Test";s:17:"Unserializationhere!";}’
unserialize($value);
```

## PHP Autoloading

-Later-


## Object Instantiation

-Later-

## Exploit the vulnerable code

-Later-


## Patch the vulnerable code

-Later-


**To Be Continue**

References : 

- https://www.notsosecure.com/remote-code-execution-via-php-unserialize/
- https://securitycafe.ro/2015/01/05/understanding-php-object-injection/
- https://www.owasp.org/index.php/PHP_Object_Injection