---
layout: post
title: "Writeup Gemastik 2016 : Java Authentication - 50"
description: "Writeup Gemastik 2016 Online Capture The Flag Qualification"
headline: 
modified: 2016-08-15
category: gemastik,ctf
tags: [gemastik,ctf]
imagefeature: /images/gemastik.png
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

![18 Duel Maut](/images/java-authentication.png)


### Solusi :

Decompile file soal `Authentication.class` bisa menggunakan JD-GUI ( Desktop Application ) atau menggunakan online decompiler
kita coba pake [online decompiler](http://www.javadecompilers.com/) 

```java
        if (string.equals("administrator") && Authentication.getHash(string2).equals("f6edb40dbd5b0568edc693c1a6bdb18e")) {
            BufferedReader bufferedReader2 = new BufferedReader(new FileReader("Authentication.flag"));
            System.out.println(bufferedReader2.readLine());
        } 
```

Terlihat authentikasi nya membandingan username dengan "administrator" dan hash(string2) atau "password" = f6edb40dbd5b0568edc693c1a6bdb18e 
kemudian decrypt md5 dengan https://hashkiller.co.uk/md5-decrypter.aspx 

`f6edb40dbd5b0568edc693c1a6bdb18e MD5 : j4v47`

```
abdilahrf:~/workspace $ nc target.netsec.gemastik.ui.ac.id 13337
Java Network Authentication Service v1.0


Username : administrator
Password : j4v47
GEMASTIK{try_to_obfuscate_Java_next_time}
```

`**Flag : GEMASTIK{try_to_obfuscate_Java_next_time}**`