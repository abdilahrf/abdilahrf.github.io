---
layout: post
title: "Writeup Idsecconf CTF 2016 : Signal - 150"
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

`http://128.199.96.39/`

### Solusi :

Diberikan file ELF signal 
`signal: ELF 64-bit LSB  executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=938d02e8b75b32f40e22eac2230c35d3000c8ddc, not stripped`

ketika di jalankan meminta password di argument nya
`usage: signal <password>`

Kita lakukan disassembly dengan *IDA* dan menemukan variable yang menyimpan password
dalam representasi Hexadecimal.

```
.data:0000000000601280 password        dq 62A2A01426E579Eh     ; DATA XREF: usr_1+2Dr
.data:0000000000601288                 public invalid_pass
```

dan dalam pengecekan password input kita di XOR dengan `1122334455667788h`

```
.text:0000000000400891 ; void usr_1(int)
.text:0000000000400891 usr_1           proc near               ; DATA XREF: main+69o
.text:0000000000400891
.text:0000000000400891 var_14          = dword ptr -14h
.text:0000000000400891 sig             = dword ptr -10h
.text:0000000000400891 var_C           = dword ptr -0Ch
.text:0000000000400891 var_4           = dword ptr -4
.text:0000000000400891
.text:0000000000400891                 push    rbp
.text:0000000000400892                 mov     rbp, rsp
.text:0000000000400895                 sub     rsp, 20h
.text:0000000000400899                 mov     [rbp+var_14], edi
.text:000000000040089C                 mov     [rbp+sig], 2
.text:00000000004008A3                 mov     [rbp+var_C], 0Ch
.text:00000000004008AA                 mov     rdx, cs:int_input
.text:00000000004008B1                 mov     rax, 1122334455667788h
.text:00000000004008BB                 xor     rdx, rax
.text:00000000004008BE                 mov     rax, cs:password
.text:00000000004008C5                 cmp     rdx, rax
.text:00000000004008C8                 setz    al
.text:00000000004008CB                 movzx   eax, al
.text:00000000004008CE                 mov     [rbp+var_4], eax
.text:00000000004008D1                 mov     esi, offset incorrect_password ; handler
.text:00000000004008D6                 mov     edi, 2          ; sig
.text:00000000004008DB                 call    _signal
.text:00000000004008E0                 mov     esi, offset correct_password ; handler
.text:00000000004008E5                 mov     edi, 0Ch        ; sig
.text:00000000004008EA                 call    _signal
.text:00000000004008EF                 mov     eax, [rbp+var_4]
.text:00000000004008F2                 cdqe
.text:00000000004008F4                 mov     eax, [rbp+rax*4+sig]
.text:00000000004008F8                 mov     esi, eax        ; sig
.text:00000000004008FA                 mov     edi, 0          ; pid
.text:00000000004008FF                 call    _kill
.text:0000000000400904                 nop
.text:0000000000400905                 leave
.text:0000000000400906                 retn
.text:0000000000400906 usr_1           endp
```

```
flag = 0x62A2A01426E579E ^ 0x1122334455667788
print hex(flag)
```
argument password yang di input dalam bentuk hex "1708194517082016"

![Flag Signal](/images/signal_flag.png)

dan di dapatkan flag : `flag{1708194517082016}`

