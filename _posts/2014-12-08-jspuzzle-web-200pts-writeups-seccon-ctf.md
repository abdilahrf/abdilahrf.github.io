---
id: 370
title: Jspuzzle Web 200pts Seccon CTF
date: 2014-12-08T20:44:50+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=370
permalink: /2014/12/jspuzzle-web-200pts-writeups-seccon-ctf/
category: Web Exploitation
tags: [SecconCTF]
---
Jspuzzle Web 200pts Seccon CTF

&nbsp;

<pre><code class="language-javascript">"use strict";

({ "function" : function() {
  this[ "null" ] = (new Function( "return" + "/*^_^*/" + "this" ))();
  var pattern = "^[w]$";
  var r = new RegExp( pattern );
  this[ r[ "exec" ]( pattern ) ][ "alert" ]( 1 );
}})[ "Function" [ "toLowerCase" ]() ]();</code></pre>

&nbsp;

Flagnya : SECCON{3678cbe0171c8517abeab9d20786a7390ffb602d}

buat challange ini minimal harus mengerti javascript buat menyelesaikannya :3 , hehe 😀