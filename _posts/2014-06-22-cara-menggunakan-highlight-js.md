---
id: 134
title: Cara Menggunakan Highlight.js
date: 2014-06-22T00:06:28+00:00
author: abdilahrf
layout: post
guid: http://hasnydes.us/?p=134
permalink: /2014/06/cara-menggunakan-highlight-js/
factory_shortcodes_assets:
  - 'a:1:{s:12:"sociallocker";b:1;}'
categories:
  - Web App
---
Cara menggunakan highlight.js untuk mewarnai kode di website atau blog anda <img src="https://www.hasnydes.us/wp-includes/images/smilies/simple-smile.png" alt=":)" class="wp-smiley" style="height: 1em; max-height: 1em;" />

  * pertama download hightligh.js <a title="Highlight JS" href="http://highlightjs.org/download/" target="_blank">Sini</a>

&nbsp;

<div id="attachment_135" style="width: 610px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.github.io/images/2014/06/highlight.png"><img class="wp-image-135" src="http://abdilahrf.github.io/images/2014/06/highlight.png" alt="Highlight Js" width="600" height="315" /></a>
  
  <p class="wp-caption-text">
    Highlight Js
  </p>
</div>

  * Buat Folder Latihan Yang Di Dalam nya terdiri dari folder &#8216; css &#8216;  & &#8216; js &#8216; dan buat file example.html

<div id="attachment_136" style="width: 610px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.github.io/images/2014/06/folder-latihan.png"><img class="wp-image-136" src="http://abdilahrf.github.io/images/2014/06/folder-latihan.png" alt="folder latihan" width="600" height="229" /></a>
  
  <p class="wp-caption-text">
    Latihan Syntax Highlight
  </p>
</div>

&nbsp;

Ambil googlecode.css yang ada di highlight.zip masukan ke folder css dan ambil highlight.pack.js masukan ke dalam folder js

Setelah itu isi kode ini di dalam file example.html

<div class="onp-locker-call" style="display: none;" data-lock-id="onpLock137654">
  <p>
  </p>
  
  <pre><code>Example Syntax Highlight

&lt;!DOCTYPE HTML&gt;
&lt;html&gt;
&lt;head&gt;
&lt;title&gt;Example Syntax Highlight&lt;/title&gt;
&lt;link rel="stylesheet" href="css/rainbow.css"&gt;
&lt;/head&gt;
&lt;body&gt; 
&lt;pre&gt;&lt;code&gt;
.awesome-thing {
 display: block;
 width: 100%;
}
#import
print
&lt;/code&gt;&lt;/pre&gt;
&lt;/body&gt;
&lt;script src="js/highlight.pack.js"&gt;&lt;/script&gt;
&lt;script&gt;hljs.initHighlightingOnLoad();&lt;/script&gt;
&lt;/html&gt;
</code></pre>
  
  <p>
    <div id="attachment_137" style="width: 470px" class="wp-caption aligncenter">
      <a href="http://abdilahrf.github.io/images/2014/06/script.png"><img class="size-full wp-image-137" src="http://abdilahrf.github.io/images/2014/06/script.png" alt="Script" width="460" height="370" /></a>
      
      <p class="wp-caption-text">
        Script
      </p>
    </div>
  </p>
  
  <p>
  </p>
</div>

script baris ke-5 adalah untuk memanggil csss googlecode style dari highlight.js

script baris ke-8 sampai 15 adalah kode yang akan di tampilkan menggunakan tag pre dan code

script baris ke-17 untuk memanggil javascript dari highlight.js

script baris ke-18 untuk mempersiapkan highlight.js saat page di load

hasil nya adalah seperti ini

&nbsp;

<div id="attachment_139" style="width: 427px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.github.io/images/2014/06/Untitled5.png"><img class="size-full wp-image-139" src="http://abdilahrf.github.io/images/2014/06/Untitled5.png" alt="Highlight JS" width="417" height="169" /></a>
  
  <p class="wp-caption-text">
    Highlight JS
  </p>
</div>

&nbsp;