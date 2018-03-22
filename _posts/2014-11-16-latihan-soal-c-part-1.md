---
id: 323
title: Latihan Soal C Part 1
date: 2014-11-16T00:11:13+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=323
permalink: /2014/11/latihan-soal-c-part-1/
category: Programming
tags: [C]
---
latihan soal c part 1 untuk mengasah logika coding kita harus banyak latihan dengan soal-soal maka saya coba mengerjakan soal dari online judge untuk mengasah logika untuk coding soalnya seperti berikut :

Indonesia memiliki cukup banyak pemain bulutangkis yang handal dan setiap tahunnya PBSI selalu merekrut dan membina puluhan ribu bibit muda dari seluruh nusantara. Saat ini terdapat N peserta putra dan M peserta putri yang sedang mengikuti pelatihan dasar bulutangkis yang diselenggarakan PBSI. Setiap peserta pelatihan sudah dinilai kemampuannya oleh pelatih dengan setiap nilai (rating) berada pada rentang 1 hingga 100.000. Untuk mengukur kemajuan pelatihan ini, sang pelatih ingin mengetahui ada berapa banyak cara membentuk sebuah tim yang terdiri dari 1 putra dan 1 putri (ganda campuran) sedemikian sehingga jumlah rating mereka setidaknya sebesar K.

<!--more-->

Contoh, diberikan data:

5 peserta putra:

  * Andi, dengan rating 100.
  * Budi, dengan rating 105.
  * Chandra, dengan rating 450.
  * Dedi, dengan rating 700.
  * Eko, dengan rating 1025.

3 peserta putri:

  * Lili, dengan rating 400.
  * Maya, dengan rating 500.
  * Nina, dengan rating 500.

dan K = 1200.

Dari data di atas, dapat disimpulkan ada 5 cara untuk membentuk tim dengan jumlah rating minimal 1200, yaitu:

  1. Dedi dengan Maya (700 + 500 = 1200).
  2. Dedi dengan Nina (700 + 500 = 1200).
  3. Eko dengan Lili (1025 + 400 = 1425).
  4. Eko dengan Maya (1025 + 500 = 1525).
  5. Eko dengan Nina (1025 + 500 = 1525).

Diberikan data rating N putra dan M putri yang masing-masing sudah terurut, buat program untuk menentukan ada berapa cara membentuk sebuah tim yang terdiri dari 1 putra dan 1 putri sedemikian sehingga jumlah rating mereka setidaknya sebesar K.

&nbsp;

### Input

Baris pertama berisi sebuah bilangan bulat T (T ≤ 20) yang menyatakan jumlah kasus. Setiap kasus dimulai dengan tiga bilangan bulat N, M dan K (1 ≤ N, M ≤ 40.000; 1 ≤ K ≤ 100.000) yang menyatakan banyaknya peserta putra, peserta putri, dan jumlah rating minimal yang diharapkan secara berurutan. Baris kedua berisi N buah bilangan bulat A<sub>i</sub> (1 ≤ A<sub>i</sub> ≤ 100.000; i=<small>1..N</small>) dengan A<sub>i &#8211; 1</sub> ≤ A<sub>i</sub> untuk i=<small>2..N</small>yang merepresentasikan rating semua peserta putra. Baris ketiga berisi M buah bilangan bulat B<sub>i</sub> (1 ≤ B<sub>i</sub> ≤ 100.000; i=<small>1..M</small>) dengan B<sub>i &#8211; 1</sub> ≤ B<sub>i</sub> untuk i=<small>2..M</small> yang merepresentasikan rating semua peserta putri.

### Output

Untuk setiap kasus, output dalam satu baris &#8220;<span style="font-family: 'Courier New';">Kasus #X: Y</span>&#8221; (tanpa kutip, antara titik dua &#8216;:&#8217; dan Y dipisahkan oleh tepat sebuah spasi) dengan X adalah nomor kasus dimulai dari 1 secara berurutan dan Y adalah sebuah bilangan bulat yang menyatakan banyaknya cara membentuk tim yang terdiri dari 1 putra dan 1 putri sedemikian sehingga jumlah rating mereka minimal K pada kasus tersebut.

&nbsp;

<table border="0" cellspacing="0" cellpadding="0">
  <tr>
    <td>
      <table border="0" width="600" cellspacing="1" cellpadding="4">
        <tr>
          <th align="left" bgcolor="#FFFFFF" width="250">
            Contoh input
          </th>
          
          <th align="left" bgcolor="#FFFFFF" width="250">
            Output untuk contoh input
          </th>
        </tr>
        
        <tr valign="top">
          <td bgcolor="#FFFFFF">
            <pre>3
5 3 1200
100 105 150 720 1025
400 500 500
5 6 1250
70 180 560 770 970
420 460 490 620 790 960
3 5 200
10 20 30
50 60 70 80 90
</pre>
          </td>
          
          <td bgcolor="#FFFFFF">
            <pre>Kasus #1: 5
Kasus #2: 12
Kasus #3: 0
</pre>
          </td>
        </tr>
      </table>
    </td>
  </tr>
</table>

&nbsp;

```c

#include<stdio.h>
#include<string.h>
#include<windows.h>
int t,m,k1[1001],k2[1001],n,max[20],temp[5];

int main(){
	
do{
    scanf("%d",&t);
    fflush(stdin);
}while(t > 20 || t <= 0);

for(int c=0;c<t;c++)
{
    do{
       scanf("%d %d %d",&n,&m,&max[c]);
    }while(n<=1 || n>= 40000 || m<= 1 || m>= 40000 || max[c]<=1 || max[c]>=100000);
    
    for(int i=0;i<n;i++){
        scanf("%d",&k1[i]);	
    }
    for(int i=0;i<m;i++){
        scanf("%d",&k2[i]);
    }

    for(int j=0;j<n;j++){
        for(int n=0;n<m;n++){
        if(k1[j] + k2[n] > max[c]){
            temp[c] += 1;
            }
        }
    }	
}

for(int c=0;c<t;c++){
    printf("Kasus #%d: %d\n",c+1,temp[c]);
}	
system("pause");
return 0;
}
```

&nbsp;