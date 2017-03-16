---
id: 363
title: BELAJAR BAHASA C PART 3
date: 2014-12-04T09:04:25+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=363
permalink: /2014/12/belajar-bahasa-c-part-3/
categories: Programming
tags: [C]
---
### Function & Recursif , Parsing parameter by value dan Parsing parameter by location ( reference )

di bahasa c kita juga bisa membuat function untuk mempermudah dan menghemat tenaga untuk coding karena hanya di tulis 1x dan bisa di gunakan seterusnya contohnya fungsi seperti ini

<pre><code class="language-c">// tipedata namafunction(Parameter (tipedata namaparameter),(tipedata namaparameter)){
//block statement
}
int functionCoba(int angka){
//Block statement
//Return namavariable atau angka
}</code></pre>

untuk function bisa menggunakan semua tipe data termasuk void tapi kalau void tidak dapat mengembalikan nilai dan tidak dapat menggunakan return coba kita buat function sederhana untuk menghitung selisih nilai anda dengan nilai teman

<pre><code class="language-c">#include &lt;stdio.h&gt;
int hitungSelisihNilai(int nilaiSaya,int nilaiTeman){
	int selisih = nilaiSaya - nilaiTeman;
	return selisih;
}

int main(){
int nilaiSaya,nilaiTeman;
printf("masukan nilai anda : ");
scanf("%d",&nilaiSaya);
fflush(stdin);

printf("masukan nilai teman : ");
scanf("%d",&nilaiTeman);
fflush(stdin);

printf("Selisih Nilai Anda Dengan Teman : %d ",hitungSelisihNilai(nilaiSaya,nilaiTeman));
getchar();
return 0;

}</code></pre>

dan kemudian kita buat contoh parsing parameter with value dan parsing parameter with reference

<pre><code class="language-c">void ubahNilaiByValue(int nilai){
	nilai = nilai + 11;
	
}

void ubahNilaiByRef(int *nilai){
	*nilai = *nilai + 22;
} 

int main(){
 //Parsing parameter by value 
 int number = 50;
 printf("Number Before : %d \n",number);
 ubahNilaiByValue(number);
 printf("Number After : %d \n",number);
 
 //Parsing parameter by refference / addres / location
 int number2 = 30;
 printf("Number2 Before : %d \n",number2);
 ubahNilaiByRef(&number2);
 printf("Number2 After : %d \n",number2);
}
</code></pre>

Parsing parameter by value number akan tetap 50 setelah atau sebelum di masukan ke function karena function tidak mengetahui adress dari number tersebut kecuali yang menggunakan parsing parameter by reference kita memberikan &#8220;&number2&#8221; yang berarti kita memberikan adress kita maka function dapat merubah value kita tambahan ini ada contoh untuk parsing array

<pre><code class="language-c">void printArray(int arr[] , int index){
 printf("Array ke %d Adalah %d \n",index,arr[index]);
}

int main(){
//parsing parameter array
  int numb_arr[] = {10,15,20,25,30};
  printArray(numb_arr,0); // Maka akan keluar array index ke [0] = 10
  getchar();  
  return 0;
}</code></pre>

Kemudian contoh rekrusif function yaitu function yang dapat memanggil dirinya sendiri di dalam function itu

<pre><code class="language-c">int faktorial(int angka){

	if(angka &lt;= 1)
		return 1;
	else
		return angka * faktorial(angka-1);
}

int main(){
	printf("Faktorial dari 6 : %d\n",faktorial(6));
        getchar();
        return 0;
}</code></pre>

&nbsp;

dan lagi ada contoh rekrusif function yaitu untuk menyelesaikan fibonnaci

<pre><code class="language-c">//fibonaci
// 1 1 2 3 5 8 13 21
int fibonaci(int angka){
	if(angka &lt;=2 )
		return 1;
	else
		fibonaci(angka-1);//1
		return fibonaci(angka-1) + fibonaci(angka-2); //4
}
int totalFibo(int angka){
		if(angka &lt;=1)
			return 1;
		else
		return fibonaci(angka)+totalFibo(angka-1); 
}

int main(){
	printf("Fibonacci dari 8 : %d\n",fibonaci(8));
	printf("Jumlah Fibonacci dari 5 : %d \n",totalFibo(4));
	getchar();
	return 0;
} </code></pre>

&nbsp;

&nbsp;

&nbsp;