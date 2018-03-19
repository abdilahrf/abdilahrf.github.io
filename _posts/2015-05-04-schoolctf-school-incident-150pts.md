---
id: 562
title: 'SchoolCTF : School Incident 150pts'
date: 2015-05-04T00:34:45+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=562
permalink: /2015/05/schoolctf-school-incident-150pts/
category: Web Exploitation
tags: [SchoolCTF]
---
> ### SchoolCTF : School Incident 150pts
> 
> Somebody called Johnny Droptables managed to infiltrate to the school schedule server. He commented this event on a forum:
> 
> LOL, look at these stupid admins, bros! They set easy-breasy root password and store md5 of the password in /etc/shadow:
> 
> > <pre><code class="language-apacheconf">root@school:#cat /etc/shadow
root:f505f1e49e98cea82c9c4265262444f6:14425:0:99999:7:::
teacher:*:14425:0:99999:7:::
adm:*:14425:0:99999:7:::
...</code></pre>
> 
> No more lessons 2day, rofl xD
> 
> Flag &#8211; root password

<!--more-->

hasil decode md5 adalah flagnya

<div id="attachment_563" style="width: 1061px" class="wp-caption aligncenter">
  <a href="http://abdilahrf.github.io/images/2015/05/flag3.png"><img class="size-full wp-image-563" src="http://abdilahrf.github.io/images/2015/05/flag3.png" alt="SchoolCTF : School Incident" width="1051" height="77" /></a>
  
  <p class="wp-caption-text">
    Flag
  </p>
</div>

Flag : **i\_am\_admin**