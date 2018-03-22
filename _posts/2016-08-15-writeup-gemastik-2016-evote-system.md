---
layout: post
title: "Gemastik CTF 2016 : Evote System - 150"
description: "Writeup Gemastik 2016 Online Capture The Flag Qualification"
headline: 
modified: 2016-08-15
date: 2016-08-15
category: CTF
tags: [gemastik,Web Exploitation]
imagefeature: /images/gemastik.png
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

![Evote System](/images/evote-system.png)


### Solusi :

Pada soal sebelumnya Intranet Maintenance yang menyinggung tentang backup file yang diedit dengan GEDIT dan db.php.save

Pada soal E-vote tanpa pikir lama kami langsung mencoba :
https://target.netsec.gemastik.ui.ac.id/3a8c9e41d7f09e76e058147a200f2229/e-vote-system/index.php.save

Ternyata di sana ada flag yang valid : 
Flag : `GEMASTIK{haXing_the_presidential_3l3ct10n_is_r34l}`

Cara yang di harapkan adaalah 
`download .git > analisa source > overwrite via extract($_POST)`

