---
id: 458
title: Reverseit Binary Seccon CTF 100pts
date: 2014-12-21T15:59:17+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=458
permalink: /2014/12/reverseit-binary-seccon-ctf-100pts/
factory_shortcodes_assets:
  - 'a:0:{}'
categories:
  - seccon
---
Reverseit Binary 100pts Seccon CTF

> [`Reverseit`](https://github.com/ctfs/write-ups/blob/master/seccon-ctf-2014/reverse-it/Reverseit)

Di Challange Ini kita di berikan file binary , kalau di teliti file itu mengandung signature yang di kenal sebagai jpeg image

> <pre class="de1">$ <span class="kw2">file</span> Reverseit
Reverseit: data
$ <span class="kw2">hexdump</span> <span class="re5">-C</span> Reverseit
00000000  9d ff <span class="nu0">70</span> 0d b6 da <span class="kw3">fc</span> <span class="nu0">93</span>  <span class="nu0">72</span> <span class="nu0">63</span> <span class="nu0">28</span> <span class="nu0">22</span> <span class="nu0">22</span> bd d2 <span class="nu0">18</span>  <span class="sy0">|</span>..p.....rc<span class="br0">(</span><span class="st0">""</span>...<span class="sy0">|</span>
...
000013b0  02 02 02 02 02 02 02 02  02 02 02 02 02 02 02 02  <span class="sy0">|</span>................<span class="sy0">|</span>
<span class="sy0">*</span>
00001ba0  02 02 02 02 02 02 02 02  02 02 02 02 02 e3 <span class="nu0">16</span> <span class="nu0">47</span>  <span class="sy0">|</span>...............G<span class="sy0">|</span>
...
00001e00  <span class="nu0">84</span> 00 <span class="nu0">84</span> 00 <span class="nu0">10</span> <span class="nu0">10</span> <span class="nu0">10</span> 00  <span class="nu0">64</span> <span class="nu0">94</span> <span class="nu0">64</span> a4 01 00 0e <strong>ff</strong>  <span class="sy0">|</span>........d.d.....<span class="sy0">|</span>
00001e10  <strong>8d ff       
</strong></pre>

Kita Gunakan script python untuk reverse keseluruhan dari file Reverseit

<!--more-->

<div class="wp-geshi-highlight-wrap5">
  <div class="wp-geshi-highlight-wrap4">
    <div class="wp-geshi-highlight-wrap3">
      <div class="wp-geshi-highlight-wrap2">
        <div class="wp-geshi-highlight-wrap">
          <div class="wp-geshi-highlight">
            <div class="python">
              <blockquote>
                <pre class="de1"><span class="kw1">import</span> <span class="kw3">sys</span>
 
<span class="kw1">def</span> swap_nibbles<span class="br0">(</span>byte<span class="br0">)</span>:
  <span class="kw1">return</span> <span class="br0">(</span><span class="br0">(</span>byte<span class="sy0">&lt;&lt;</span><span class="nu0">4</span><span class="br0">)</span> | <span class="br0">(</span>byte<span class="sy0">&gt;&gt;</span><span class="nu0">4</span><span class="br0">)</span><span class="br0">)</span> & <span class="nu0">0xff</span>
 
i<span class="sy0">=</span><span class="kw2">open</span><span class="br0">(</span><span class="kw3">sys</span>.<span class="me1">argv</span><span class="br0">[</span><span class="nu0">1</span><span class="br0">]</span><span class="sy0">,</span> <span class="st0">'rb'</span><span class="br0">)</span>
o<span class="sy0">=</span><span class="kw2">open</span><span class="br0">(</span><span class="kw3">sys</span>.<span class="me1">argv</span><span class="br0">[</span><span class="nu0">1</span><span class="br0">]</span> + <span class="st0">'.rev'</span><span class="sy0">,</span> <span class="st0">'wb'</span><span class="br0">)</span>
o.<span class="me1">write</span><span class="br0">(</span><span class="st0">''</span>.<span class="me1">join</span><span class="br0">(</span><span class="kw2">map</span><span class="br0">(</span><span class="kw2">chr</span><span class="sy0">,</span> <span class="kw2">map</span><span class="br0">(</span>swap_nibbles<span class="sy0">,</span> <span class="kw2">map</span><span class="br0">(</span><span class="kw2">ord</span><span class="sy0">,</span> i.<span class="me1">read</span><span class="br0">(</span><span class="br0">)</span><span class="br0">[</span>::-<span class="nu0">1</span><span class="br0">]</span><span class="br0">)</span><span class="br0">)</span><span class="br0">)</span><span class="br0">)</span><span class="br0">)</span>
i.<span class="me1">close</span><span class="br0">(</span><span class="br0">)</span>
o.<span class="me1">close</span><span class="br0">(</span><span class="br0">)</span></pre>
              </blockquote>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

> <div class="wp-geshi-highlight-wrap5">
>   <div class="wp-geshi-highlight-wrap4">
>     <div class="wp-geshi-highlight-wrap3">
>       <div class="wp-geshi-highlight-wrap2">
>         <div class="wp-geshi-highlight-wrap">
>           <div class="wp-geshi-highlight">
>             <div class="bash">
>               <pre class="de1">$ python rev_bytes.py Reverseit
$ <span class="kw2">file</span> Reverseit.rev 
Reverseit.rev: JPEG image data, JFIF standard <span class="nu0">1.01</span>
$ <span class="kw2">hexdump</span> <span class="re5">-C</span> Reverseit.rev
00000000  ff d8 ff e0 00 <span class="nu0">10</span> 4a <span class="nu0">46</span>  <span class="nu0">49</span> <span class="nu0">46</span> 00 01 01 01 00 <span class="nu0">48</span>  <span class="sy0">|</span>......JFIF.....H<span class="sy0">|</span>
...</pre>
>             </div>
>           </div>
>         </div>
>       </div>
>     </div>
>   </div>
> </div>
> 
> Hasilnya Reveseit.rev adalah JPEG Seperti Ini , Masih terbalik tulisannya
  
> [<img class="aligncenter size-full wp-image-742" src="http://blogs.univ-poitiers.fr/e-laize/files/2014/12/Reverseit.jpg" alt="Reverseit.rev" width="200" height="26" />](http://blogs.univ-poitiers.fr/e-laize/files/2014/12/Reverseit.jpg)Agar Normal mari kita code untuk meluruskannya :3

<div class="wp-geshi-highlight-wrap5">
  <div class="wp-geshi-highlight-wrap4">
    <div class="wp-geshi-highlight-wrap3">
      <div class="wp-geshi-highlight-wrap2">
        <div class="wp-geshi-highlight-wrap">
          <div class="wp-geshi-highlight">
            <div class="python">
              <blockquote>
                <pre class="de1"><span class="kw1">import</span> <span class="kw3">sys</span>
<span class="kw1">from</span> PIL <span class="kw1">import</span> Image
 
i <span class="sy0">=</span> Image.<span class="kw2">open</span><span class="br0">(</span><span class="kw3">sys</span>.<span class="me1">argv</span><span class="br0">[</span><span class="nu0">1</span><span class="br0">]</span><span class="br0">)</span>
o <span class="sy0">=</span> Image.<span class="kw3">new</span><span class="br0">(</span>i.<span class="me1">mode</span><span class="sy0">,</span> i.<span class="me1">size</span><span class="br0">)</span>
idata<span class="sy0">,</span> odata <span class="sy0">=</span> i.<span class="me1">load</span><span class="br0">(</span><span class="br0">)</span><span class="sy0">,</span> o.<span class="me1">load</span><span class="br0">(</span><span class="br0">)</span>
 
<span class="kw1">for</span> y <span class="kw1">in</span> <span class="kw2">range</span><span class="br0">(</span>i.<span class="me1">size</span><span class="br0">[</span><span class="nu0">1</span><span class="br0">]</span><span class="br0">)</span>:
  <span class="kw1">for</span> x <span class="kw1">in</span> <span class="kw2">range</span><span class="br0">(</span>i.<span class="me1">size</span><span class="br0">[</span><span class="nu0"></span><span class="br0">]</span><span class="br0">)</span>:
    odata<span class="br0">[</span>x<span class="sy0">,</span> y<span class="br0">]</span> <span class="sy0">=</span> idata<span class="br0">[</span>i.<span class="me1">size</span><span class="br0">[</span><span class="nu0"></span><span class="br0">]</span>-x-<span class="nu0">1</span><span class="sy0">,</span> y<span class="br0">]</span>
 
o.<span class="me1">save</span><span class="br0">(</span><span class="kw3">sys</span>.<span class="me1">argv</span><span class="br0">[</span><span class="nu0">1</span><span class="br0">]</span> + <span class="st0">'.jpg'</span><span class="br0">)</span></pre>
              </blockquote>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

<div class="wp-geshi-highlight-wrap5">
  <div class="wp-geshi-highlight-wrap4">
    <div class="wp-geshi-highlight-wrap3">
      <div class="wp-geshi-highlight-wrap2">
        <div class="wp-geshi-highlight-wrap">
          <div class="wp-geshi-highlight">
            <div class="bash">
              <blockquote>
                <pre class="de1"><span class="co4">$ </span>python revh_image.py Reverseit.rev</pre>
              </blockquote>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

dan kita mendapatkan gambar yang normal bertulisan `SECCON{6in_tex7}`:
  
[<img class="aligncenter size-full wp-image-743" src="http://blogs.univ-poitiers.fr/e-laize/files/2014/12/Reverseit.rev_.jpg" alt="Reverseit.rev.jpg" width="200" height="26" />](http://blogs.univ-poitiers.fr/e-laize/files/2014/12/Reverseit.rev_.jpg)

dari coding ruby di atas , dengan command ini kamu juga bisa melakukan reverse :

<div class="wp-geshi-highlight-wrap5">
  <div class="wp-geshi-highlight-wrap4">
    <div class="wp-geshi-highlight-wrap3">
      <div class="wp-geshi-highlight-wrap2">
        <div class="wp-geshi-highlight-wrap">
          <div class="wp-geshi-highlight">
            <div class="bash">
              <blockquote>
                <pre class="de1">$ <span class="kw2">hexdump</span> <span class="re5">-ve</span> <span class="st_h">'1/1 "%.2x"'</span> Reverseit <span class="sy0">|</span> <span class="kw2">rev</span> <span class="sy0">|</span> xxd <span class="re5">-r</span> <span class="re5">-p</span> <span class="sy0">&gt;</span> Reverseit.rev
$ convert <span class="re5">-flop</span> Reverseit.rev Reverseit.rev.jpg</pre>
              </blockquote>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

&nbsp;