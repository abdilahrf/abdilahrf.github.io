---
layout: post
title: "Writeup Idsecconf CTF 2016 : Bit - 100"
description: "Writeup Idsecconf CTF 2016 Online"
headline: 
modified: 2016-08-29
category: idsecconf
tags: [idsecconf,ctf,reverse]
imagefeature: /images/idsecconf.jpg
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

`nc 128.199.232.109 17846`

### Solusi :

Diketahui bit adalah ELF x64 yang stripped
`bit: ELF 64-bit LSB  executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=18a69419634b377f3ffbb736952951340bcd045b, stripped`

Hasil objdump menunjukan file bit menggunakan beberapa lib

abdilahrf:~/workspace/CTF/idsecconf/reverse/bit $ objdump -T bit                                                    

bit:     file format elf64-x86-64

```objdump

DYNAMIC SYMBOL TABLE:
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 putchar
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 puts
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.4   __stack_chk_fail
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 printf
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 read
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 __libc_start_main
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 srand
0000000000000000  w   D  *UND*  0000000000000000              __gmon_start__
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 time
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 fflush
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 open
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.7   __isoc99_scanf
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 sleep
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 rand
00000000006020a0 g    DO .bss   0000000000000008  GLIBC_2.2.5 stdout
```

Terlihat rand,time,srand karena ini random number guessing yang saya pikirkan adalah
vulnerability PRNG seed tapi ternyata bukan lagi pula di situ jika kita menebak dengan benar
maka akan di hadiahkan jumlah taruhan + 100 programnya juga tidak melakukan looping maka
akan mustahil untuk menebak dengan benar sampai BTC nya = 100000

```cpp
__int64 __fastcall sub_4009BB(__int64 a1, __int64 a2)
{
  unsigned int v2; // eax@1
  signed __int64 v3; // rbx@1
  __int64 result; // rax@5
  signed int v5; // [sp+14h] [bp-2Ch]@7
  __int64 v6; // [sp+18h] [bp-28h]@3
  __int64 v7; // [sp+20h] [bp-20h]@3
  __int64 v8; // [sp+28h] [bp-18h]@1

  v2 = time(0LL);
  srand(v2);
  v3 = (signed __int64)rand() << 32;
  v8 = v3 | rand();
  if ( v8 > 13 )
    v8 += 14LL;
  puts("++++ BITCOIN KASINO ++++!");
  fflush(0LL);
  sleep(3u);
  puts("Eddy Tongsis Masuk Dengan BTC90000...\nGayus Timbunan Masuk Dengan BTC100000...\nDan Kamu Dengan Saldo 100BTC.");
  fflush(0LL);
  sleep(3u);
  puts("Tebak Angka Yang Akan Naik - (Diatas 13)");
  fflush(0LL);
  sleep(3u);
  printf("Masukkan Angka Keberuntunganmu: ", a2);
  fflush(0LL);
  __isoc99_scanf(4197721LL, &v6);
  printf("Nilai Taruhanmu (BTC): ");
  fflush(0LL);
  __isoc99_scanf(4197750LL, &v7);
  if ( (signed int)v7 <= 100 && v7 >= 0 )
  {
    sub_4008CD(8LL);
    puts("Hasilnya adalah...");
    fflush(0LL);
    sleep(3u);
    if ( v6 == v8 )
    {
      puts("Ajaib! Selamat Atas Tebakannya.");
      v5 = v7 + 100;
    }
    else
    {
      v5 = 100 - v7;
      printf("Bukan %lld. Yang Benar Adalah %lld.\n", v6, v8);
    }
    fflush(0LL);
    sleep(3u);
    if ( v5 > 100000 )
    {
      sub_4008CD(5LL);
      printf("Anda Memenangkan %dBTC\nSelain Itu Akan Kami Berikan Bonus Berupa\n", (unsigned int)v5);
      sleep(5u);
      sub_40091C(5LL);
    }
    else
    {
      printf("Kamu Kalah! (%dBTC) \n", (unsigned int)v5);
    }
    result = 0LL;
  }
  else
  {
    puts("Taruhan ditolak, saldo kamu hanya 100BTC");
    result = 0LL;
  }
  return result;
}
```

Idenya adalah kita tidak harus menebak angka yang random tapi kita bisa mengoverflow 
saldo BTC kita dengan membuat angka tebakan kita overflow dan kita bisa menginput sembarang
di jumlah taruhan.

variable yang menyimpan saldo BTC dan angka keberuntungan diketahui int 64.
setelah membaca tentang Int64 di [Integer Overflow](https://rosettacode.org/wiki/Integer_overflow)

maka kita dapat melakukan integer overflow sehingga BTC kita > 10000

```bash
echo -n -e "18446744073709551616\n3133731337\n"| ./bit 
```

![Bit Flag Lokal](/images/bit_flaglokal.png)

dan di dapatkan flag : `flag{Integer_H4ri_Ini}`

![Bit Flag Remote](/images/bit_flagremote.png)

