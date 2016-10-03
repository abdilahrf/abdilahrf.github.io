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
$value='a:1:{s:4:"Test";s:17:"Unserializationhere!";}'
unserialize($value);
?>
```

## PHP Autoloading

Dalam php kita bisa mendifinisikan special function yang bisa di exsekusi secara otomatis
jadi untuk menjalankannya tidak perlu panggil fungsinya lagi biasanya disebut dengan php magic function atau magic method
php juga mempunyai beberapa magic method seperti __construct/__destruct.

Contoh beberapa magic function di php : __construct(), __destruct(), __call(), __callSt
atic(), __get(), __set(), __isset(), __unset(), __sleep(), __wakeup(), __toString(), __invoke(), __set_state(), __clone(), and __autoload().

Magic method : 
`Exception::__toStringErrorException::__toStringDateTime::__wakeupReflectionException::__toStringReflectionFunctionAbstract::__toStringReflectionFunction::__toStringReflectionParameter::__toStringReflectionMethod::__toStringReflectionClass::__toStringReflectionObject::__toStringReflectionProperty::__toStringReflectionExtension::__toStringLogicException::__toStringBadFunctionCallException::__toStringBadMethodCallException::__toStringDomainException::__toStringInvalidArgumentException::__toStringLengthException::__toStringOutOfRangeException::__toStringRuntimeException::__toString`

Ref: http://www.programmerinterview.com/index.php/php-questions/php-what-are-magic-functions/



## Object Instantiation

Instantiation adalah ketika class menjadi objek saat kita buat instance dari class tersebut
contohnya ketika kita buat `new namaClass()` , namaClass() jadi instantiated object

## Exploit the vulnerable code

```php
<?php

class foo {
	public $file = "text.txt";
	public $data = "stringnya";

	// php magic function
	function __destruct(){
		file_put_contents($this->file, $this->data);
	}
}

$file_name = $_GET['session_filename'];

print "Readfile " . $file_name . "<br>";
if(!file_exists($file_name)){
	print "no file\n";
}else{
	unserialize(file_get_contents($file_name));
}

?>
```

pertama kita lihat apakah class yang vulnerable memanggil magic function dari php, 
jika iya berarti kita mungkin untuk mengubah flow dari applikasi tersebut.

code vulnerable di atas memanggil `__destruct()`

```php
<?php
class foo {
	public $file = "text.txt";
	public $data = "stringnya";

	// php magic function
	function __destruct(){
		file_put_contents($this->file, $this->data);
	}
}

```

#### Racik payload

`O:3:%22foo%22:2:{s:4:%22file%22;s:9:%22shell.php%22;s:4:%22data%22;s:5:%22aaaa%22;}`

- `O:3:`                            : Object, terima 3 parameter
- `"foo":2:{`                       : Parameter foo terima 2 value
- `s:4:"file";s:9:"shell.php";`     : String, 4 karakter "file",String 9 karakter "shell.php" 
- `s:4:"data";s:5:"aaaa";}`         : String, 4 karakter "data",String 5 karakter "aaaaa" 

Ketika input string kita di unserialized maka kita dapat mengontrol properti dar class "foo".
Di dalam magic method __destruct maka akan menggunakan
value yang sudah kita kontrol dan akan membuat file shell.php dengan isi "aaaaa".


## Patch the vulnerable code

Lakukan validasi terhadap user input. dan gunakan json untuk pertukaran data.
bila memang harus menggunakan external serialize bisa juga tambahkan validasi dengan hmac()
atau lain sebagainya agar tidak ada input yang bisa mengexploitasi sistem.



References : 

- https://www.notsosecure.com/remote-code-execution-via-php-unserialize/
- https://securitycafe.ro/2015/01/05/understanding-php-object-injection/
- http://www.inulledmyself.com/2015/02/exploiting-memory-corruption-bugs-in.html
- https://www.owasp.org/index.php/PHP_Object_Injection