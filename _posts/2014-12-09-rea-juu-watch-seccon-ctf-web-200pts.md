---
id: 408
title: 'Seccon CTF 2014: REA-JUU WATCH  Web 200pts'
date: 2014-12-09T09:38:02+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=408
permalink: /2014/12/rea-juu-watch-seccon-ctf-web-200pts/
category: ctf
tags: [SecconCTF,Web Exploitation]
---
<span style="color: #ffffff;"><strong>REA-JUU</strong> <strong>WATCH</strong> <strong>Seccon</strong> <strong>CTF</strong></span>

### REA-JUU WATCH Seccon CTF Web 200pts

> <http://reajuu.pwn.seccon.jp/>

kunci dari menyelesaikan soal ini ada di source codenya , untuk menyelesaikannya kita login dulu dengan user yang di dapatkan dari create new account . selesaikan sampai mendapatkan point , pilihannya bebas

[<img class="aligncenter wp-image-409" src="http://abdilahrf.github.io/images/2014/12/Untitled-1024x332.png" alt="Untitled" width="700" height="227" />](http://abdilahrf.github.io/images/2014/12/Untitled.png)

&nbsp;

kalau sudah sampai bagian itu kemudian view source , kemudian kita dapat fungsi yang menarik<!--more-->

<pre><code class="language-javascript">&lt;script&gt;
function finishpoint(){
	$.getJSON("/users/chk/2138", null, function(data){
		point = data.point;
		$("#finishpoint").text("Your score is " + point + "point!");
	});
}
&lt;/script&gt;</code></pre>

parameter _/users/chk/2138  _di gunakan untuk mendapatkan score si user dengan id 2138

coba kita buka parameternya melalui url

> {&#8220;username&#8221;:&#8221;qk198nv9&#8243;,&#8221;password&#8221;:&#8221;3ebdosx2&#8243;,&#8221;point&#8221;:-31}

hasilnya muncul username , password dan point kita untuk mendapatkan admin yang di id=1 _/users/chk/1_

> {&#8220;username&#8221;:&#8221;rea-juu&#8221;,&#8221;password&#8221;:&#8221;way\_t0\_f1ag&#8221;,&#8221;point&#8221;:99999}

nah kita dapat username dan password adminnya , kita coba bermain sampai selesai lagi dengan id tersebut dan liat point nya

[<img class="aligncenter wp-image-410" src="http://abdilahrf.github.io/images/2014/12/Untitled1.png" alt="Untitled" width="828" height="197" />](http://abdilahrf.github.io/images/2014/12/Untitled1.png)

&nbsp;

> Flag : SECCON{REA\_JUU\_Ji8A_NYAN}