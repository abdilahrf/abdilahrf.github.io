---
id: 334
title: Latihan Soal C Part 2
date: 2014-11-17T18:42:01+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=334
permalink: /2014/11/latihan-soal-c-part-2/
category: Programming
tags: [C]
---
Lanjut lagi latihan kita dengan bahasa C &#8230; soal ini sebenarnya menimplementasikan nested for dengan selection sehingga menjadi kotak dengan dalam yang kosong soalnya seperti berikut

[<img class="aligncenter wp-image-335" src="http://abdilahrf.github.io/images/2014/11/soal10-1024x415.png" alt="soal10" width="1200" height="487" />](http://abdilahrf.github.io/images/2014/11/soal10.png)

```c
#include<stdio.h>

int main(){
	int x; // mendifinisikan var x dengan int
	scanf("%d",&x); // membuat input field
	fflush(stdin);// untuk menyimpan buffer input
	// menggunakan nestedfor untuk membuat kotak
	for(int i=0;i&lt;x;i++){ // for pertama untuk baris kebawah
		for(int j=0;j&lt;x;j++){ // for kedua untuk kekanan
			if(i==0 || i==4 || j==0 || j==4){ // validasi jika di baris tertentu munculkan *
			printf("*"); // kondisi benar munculkan * 
			}else{ // kondisi else 
			printf(" "); // munculkan " "
			} 
		} // tutup for kedua
			printf("\n"); // \n untuk memberikan enter agar menjadi 5 baris
	} // tutup for pertama
	getchar(); // mendapatkan char " "
	return 0; 
}
```

[<img class="aligncenter size-full wp-image-336" src="http://abdilahrf.github.io/images/2014/11/test.png" alt="test" width="679" height="339" />](http://abdilahrf.github.io/images/2014/11/test.png)