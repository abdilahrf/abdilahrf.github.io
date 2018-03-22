---
id: 455
title: 'Seccon CTF 2014: Get The Key - Forensic 100pts'
date: 2014-12-21T15:43:58+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=455
permalink: /2014/12/get-the-key-seccon-ctf-2014-100pts/
category: CTF
tags: [SecconCTF,Forensic]
---
Get The Key Seccon CTF 2014 100pts

Download File Soal

> `<a href="https://github.com/ctfs/write-ups/blob/master/seccon-ctf-2014/get-the-key/nw100.pcap">nw100.pcap</a>`

Kita diberikan hasil rekam network dengan wireshark file format .pcap , buka file tersebut dengan wireshark untuk memulai menganalisa kemudian di sana kita menemukan ada akses ke http://133.242.224.21:6809/nw100/ menggunakan username : seccon2014 dan password : YourBattleField

Menggunakan curl kita coba login ke ip tersebut

> <pre>$ curl --user <span class="pl-s1"><span class="pl-pds">'</span>seccon2014:YourBattleField<span class="pl-pds">'</span></span> <span class="pl-s1"><span class="pl-pds">'</span>http://133.242.224.21:6809/nw100/<span class="pl-pds">'</span><span class="pl-pds">'</span></span>
<span class="pl-s1">&lt;!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN"&gt;</span>
<span class="pl-s1">&lt;html&gt;</span>
<span class="pl-s1"> &lt;head&gt;</span>
<span class="pl-s1">  &lt;title&gt;Index of /nw100&lt;/title&gt;</span>
<span class="pl-s1"> &lt;/head&gt;</span>
<span class="pl-s1"> &lt;body&gt;</span>
<span class="pl-s1">&lt;h1&gt;Index of /nw100&lt;/h1&gt;</span>
<span class="pl-s1">&lt;table&gt;&lt;tr&gt;&lt;th&gt;&lt;img src="/icons/blank.gif" alt="[ICO]"&gt;&lt;/th&gt;&lt;th&gt;&lt;a href="?C=N;O=D"&gt;Name&lt;/a&gt;&lt;/th&gt;&lt;th&gt;&lt;a href="?C=M;O=A"&gt;Last modified&lt;/a&gt;&lt;/th&gt;&lt;th&gt;&lt;a href="?C=S;O=A"&gt;Size&lt;/a&gt;&lt;/th&gt;&lt;th&gt;&lt;a href="?C=D;O=A"&gt;Description&lt;/a&gt;&lt;/th&gt;&lt;/tr&gt;&lt;tr&gt;&lt;th colspan="5"&gt;&lt;hr&gt;&lt;/th&gt;&lt;/tr&gt;</span>
<span class="pl-s1">&lt;tr&gt;&lt;td valign="top"&gt;&lt;img src="/icons/back.gif" alt="[DIR]"&gt;&lt;/td&gt;&lt;td&gt;&lt;a href="/"&gt;Parent Directory&lt;/a&gt;&lt;/td&gt;&lt;td&gt;&nbsp;&lt;/td&gt;&lt;td align="right"&gt;  - &lt;/td&gt;&lt;td&gt;&nbsp;&lt;/td&gt;&lt;/tr&gt;</span>
<span class="pl-s1">&lt;tr&gt;&lt;td valign="top"&gt;&lt;img src="/icons/text.gif" alt="[TXT]"&gt;&lt;/td&gt;&lt;td&gt;&lt;a href="key.html"&gt;key.html&lt;/a&gt;&lt;/td&gt;&lt;td align="right"&gt;29-Nov-2014 22:12  &lt;/td&gt;&lt;td align="right"&gt; 48 &lt;/td&gt;&lt;td&gt;&nbsp;&lt;/td&gt;&lt;/tr&gt;</span>
<span class="pl-s1">&lt;tr&gt;&lt;th colspan="5"&gt;&lt;hr&gt;&lt;/th&gt;&lt;/tr&gt;</span>
<span class="pl-s1">&lt;/table&gt;</span>
<span class="pl-s1">&lt;address&gt;Apache/2.2.22 (Debian) Server at 133.242.224.21 Port 6809&lt;/address&gt;</span>
<span class="pl-s1">&lt;/body&gt;&lt;/html&gt;</span></pre>

Terlihat ada file key.html , Coba kita lihat isinya

> <pre>curl --user <span class="pl-s1"><span class="pl-pds">'</span>seccon2014:YourBattleField<span class="pl-pds">'</span></span> <span class="pl-s1"><span class="pl-pds">'</span>http://133.242.224.21:6809/nw100/key.html<span class="pl-pds">'</span></span>
<span class="pl-k">&lt;</span>HTML<span class="pl-k">&gt;</span>
SECCON{Basic_NW_Challenge_Done<span class="pl-k">!</span>}
<span class="pl-k">&lt;</span>/HTML<span class="pl-k">&gt;</span></pre>

Ternyata disana ada flagnya :

<pre>SECCON{Basic_NW_Challenge_Done<span class="pl-k">!</span>}</pre>