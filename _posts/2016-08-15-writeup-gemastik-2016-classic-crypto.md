---
layout: post
title: "Writeup Gemastik 2016 : Classic Crypto - 50"
description: "Writeup Gemastik 2016 Online Capture The Flag Qualification"
headline: 
modified: 2016-08-15
date: 2016-08-15
category: Crypto
tags: [gemastik]
imagefeature: /images/gemastik.png
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

![Classic Crypto](/images/classic-crypto.png)


### Solusi :

```python
def decrypt(text):
  value = 13
  clearText = ""

  for c in text:
    cipherChar = rot(c, value)
    value = (value - 3 ) % 26
    clearText += cipherChar

  print reverse(clearText)
```

```
abdilahrf:~/workspace/CTF/gemastik $ python classic.py 
=== Gemastik - Classic Crypto ===


Insert your text :
}h3dokh_yfvxlm_zdaqs_lselv_k_aqqkmm_iyepli_oxknymg_unukx_qy_yfryi{NOCEPEZE
GEMASTIK{learn_to_break_classic_cipher_before_u_learn_about_modern_ciph3r}
Encrypted text :
```