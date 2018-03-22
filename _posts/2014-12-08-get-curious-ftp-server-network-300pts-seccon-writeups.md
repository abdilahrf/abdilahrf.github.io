---
id: 374
title: 'Seccon CTF 2014: Get curious &#8220;FTP&#8221; server - Network 300pts'
date: 2014-12-08T20:52:07+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=374
permalink: /2014/12/get-curious-ftp-server-network-300pts-seccon-writeups/
category: CTF
tags: [SecconCTF,Misc]
---
### Get curious FTP server Network 300pts Seccon Writeups

Login Ftpnya ternyata anonymous user bisa bebas login 😀

ftp ftpsv.quals.seccon.jp

masukin asal password atau kosongkan , trus aktiv kan mode passive

<pre><code class="language-markup">ftp&gt; passive
Passive mode on.

</code></pre>

Untuk liat list directory pake  quote “STAT .”

<pre><code class="language-markup">ftp&gt; quote "STAT ."
213-Status follows:
drwxr-xr-x    2 0        107          4096 Nov 29 04:43 .
drwxr-xr-x    2 0        107          4096 Nov 29 04:43 ..
-rw-r--r--    1 0        0              38 Nov 29 04:43 key_is_in_this_file_afjoirefjort94dv7u.txt
</code></pre>

Untuk download file key nya

<pre><code class="language-markup">ftp&gt; get key_is_in_this_file_afjoirefjort94dv7u.txt
local: key_is_in_this_file_afjoirefjort94dv7u.txt 
remote: key_is_in_this_file_afjoirefjort94dv7u.txt227 
Entering Passive Mode (133,242,224,21,233,51).150 
Opening BINARY mode data connection for key_is_in_this_file_afjoirefjort94dv7u.txt (38 bytes).226 Transfer complete.</code></pre>

<pre>cat key_is_in_this_file_afjoirefjort94dv7u.txt  &gt;&gt; SECCON{S0m3+im3_Pr0t0c0l_t411_4_1i3.}</pre>