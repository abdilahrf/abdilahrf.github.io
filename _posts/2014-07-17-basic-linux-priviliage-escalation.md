---
id: 177
title: Basic Linux Privilage Escalation
date: 2014-07-17T13:42:42+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=177
permalink: /2014/07/basic-linux-privilage-escalation/
category: Privilage Escalation
tags: [Linux]
---
<img class="alignnone" src="http://linoxide.com/guide/linux-cheat-sheet.png" alt="" width="1218" height="2038" />
  
<!--more-->

**untuk mendapatkan informasi os**

<pre>cat /etc/issue
cat /etc/*-release
  cat /etc/lsb-release      # Debian based
  cat /etc/redhat-release   # Redhat based
</pre>

**
  
Check The Kernel**

<pre>cat /proc/version
uname -a
uname -mrs
rpm -q kernel
dmesg | grep Linux
ls /boot | grep vmlinuz-
</pre>

**cek adanya printer atau . ?**

<pre>lpstat -a
</pre>

<!--more-->

**Cek Service yang berjalan , Cek Service denga privilage** 

<pre>ps aux
ps -ef
top
cat /etc/services
</pre>

**Service dengan root**

<pre>ps aux | grep root
ps -ef | grep root
</pre>

**Check Installed Application**

<pre>ls -alh /usr/bin/
ls -alh /sbin/
dpkg -l
rpm -qa
ls -alh /var/cache/apt/archivesO
ls -alh /var/cache/yum/
</pre>

**Cek Service setting lepas konfigurasi , apakah ada kelemahan dengan plugin yang tercantum**

<pre>cat /etc/syslog.conf
cat /etc/chttp.conf
cat /etc/lighttpd.conf
cat /etc/cups/cupsd.conf
cat /etc/inetd.conf
cat /etc/apache2/apache2.conf
cat /etc/my.conf
cat /etc/httpd/conf/httpd.conf
cat /opt/lampp/etc/httpd.conf
ls -aRl /etc/ | awk '$1 ~ /^.*r.*/
</pre>

**Check Cronjob Schedule**

<pre>crontab -l
ls -alh /var/spool/cron
ls -al /etc/ | grep cron
ls -al /etc/cron*
cat /etc/cron*
cat /etc/at.allow
cat /etc/at.deny
cat /etc/cron.allow
cat /etc/cron.deny
cat /etc/crontab
cat /etc/anacrontab
cat /var/spool/cron/crontabs/root
</pre>

**Find Plain text or passsword stored**

<pre>grep -i user [filename]
grep -i pass [filename]
grep -C 5 "password" [filename]
find . -name "*.php" -print0 | xargs -0 grep -i -n "var $password"   # Joomla
</pre>

**Cek Networking**

<pre>/sbin/ifconfig -a
cat /etc/network/interfaces
cat /etc/sysconfig/network
</pre>

**What are the network configuration settings? What can you find out about this network? DHCP server? DNS server? Gateway?**

<pre>at /etc/resolv.conf
cat /etc/sysconfig/network
cat /etc/networks
iptables -L
hostname
dnsdomainname
</pre>

**What other users & hosts are communicating with the system?**

<pre>lsof -i
lsof -i :80
grep 80 /etc/services
netstat -antup
netstat -antpx
netstat -tulpn
chkconfig --list
chkconfig --list | grep 3:on
last
w
</pre>

**Whats cached? IP and/or MAC addresses**

<pre>arp -e
route
/sbin/route -nee
</pre>

**Is packet sniffing possible? What can be seen? Listen to live traffic**

<pre>tcpdump tcp dst 192.168.1.7 80 and tcp dst 10.2.2.222 21
</pre>

**Who are you? Who is logged in? Who has been logged in? Who else is there? Who can do what?**

<pre>id
who
w
last
cat /etc/passwd | cut -d:    # List of users
grep -v -E "^#" /etc/passwd | awk -F: '$3 == 0 { print $1}'   # List of super users
awk -F: '($3 == "0") {print}' /etc/passwd   # List of super users
cat /etc/sudoers
sudo -l
</pre>

**what is sensitive file can be found**

<pre>cat /etc/passwd
cat /etc/group
cat /etc/shadow
ls -alh /var/mail/
</pre>

 **it&#8217;s possbile to acces ?**

<pre>ls -ahlR /root/
ls -ahlR /home/
</pre>

**Are there any passwords in; scripts, databases, configuration files or log files? Default paths and locations for passwords**

<pre>cat /var/apache2/config.inc
cat /var/lib/mysql/mysql/user.MYD
cat /root/anaconda-ks.cfg
</pre>

**What has the user being doing? Is there any password in plain text? What have they been edting?**

<pre>cat ~/.bash_history
cat ~/.nano_history
cat ~/.atftp_history
cat ~/.mysql_history
cat ~/.php_history
</pre>

**What user information can be found?**

<pre>cat ~/.bashrc
cat ~/.profile
cat /var/mail/root
cat /var/spool/mail/root
</pre>

**Can private-key information be found?**

<pre>cat ~/.ssh/authorized_keys
cat ~/.ssh/identity.pub
cat ~/.ssh/identity
cat ~/.ssh/id_rsa.pub
cat ~/.ssh/id_rsa
cat ~/.ssh/id_dsa.pub
cat ~/.ssh/id_dsa
cat /etc/ssh/ssh_config
cat /etc/ssh/sshd_config
cat /etc/ssh/ssh_host_dsa_key.pub
cat /etc/ssh/ssh_host_dsa_key
cat /etc/ssh/ssh_host_rsa_key.pub
cat /etc/ssh/ssh_host_rsa_key
cat /etc/ssh/ssh_host_key.pub
cat /etc/ssh/ssh_host_key
</pre>

**Which configuration files can be written in /etc/? Able to reconfigure a service?**

<pre>ls -aRl /etc/ | awk '$1 ~ /^.*w.*/' 2&gt;/dev/null     # Anyone
ls -aRl /etc/ | awk '$1 ~ /^..w/' 2&gt;/dev/null       # Owner
ls -aRl /etc/ | awk '$1 ~ /^.....w/' 2&gt;/dev/null    # Group
ls -aRl /etc/ | awk '$1 ~ /w.$/' 2&gt;/dev/null        # Other

find /etc/ -readable -type f 2&gt;/dev/null               # Anyone
find /etc/ -readable -type f -maxdepth 1 2&gt;/dev/null   # Anyone

</pre>

**What can be found in /var/ ?**

<pre>ls -alh /var/log
ls -alh /var/mail
ls -alh /var/spool
ls -alh /var/spool/lpd
ls -alh /var/lib/pgsql
ls -alh /var/lib/mysql
cat /var/lib/dhcp3/dhclient.leases
</pre>

**Any settings/files (hidden) on website? Any settings file with database information?**

<pre>ls -alhR /var/www/
ls -alhR /srv/www/htdocs/
ls -alhR /usr/local/www/apache22/data/
ls -alhR /opt/lampp/htdocs/
ls -alhR /var/www/html/
</pre>

**Is there anything log(s) files**

<pre>cat /etc/httpd/logs/access_log
cat /etc/httpd/logs/access.log
cat /etc/httpd/logs/error_log
cat /etc/httpd/logs/error.log
cat /var/log/apache2/access_log
cat /var/log/apache2/access.log
cat /var/log/apache2/error_log
cat /var/log/apache2/error.log
cat /var/log/apache/access_log
cat /var/log/apache/access.log
cat /var/log/auth.log
cat /var/log/chttp.log
cat /var/log/cups/error_log
cat /var/log/dpkg.log
cat /var/log/faillog
cat /var/log/httpd/access_log
cat /var/log/httpd/access.log
cat /var/log/httpd/error_log
cat /var/log/httpd/error.log
cat /var/log/lastlog
cat /var/log/lighttpd/access.log
cat /var/log/lighttpd/error.log
cat /var/log/lighttpd/lighttpd.access.log
cat /var/log/lighttpd/lighttpd.error.log
cat /var/log/messages
cat /var/log/secure
cat /var/log/syslog
cat /var/log/wtmp
cat /var/log/xferlog
cat /var/log/yum.log
cat /var/run/utmp
cat /var/webmin/miniserv.log
cat /var/www/logs/access_log
cat /var/www/logs/access.log
ls -alh /var/lib/dhcp3/
ls -alh /var/log/postgresql/
ls -alh /var/log/proftpd/
ls -alh /var/log/samba/

Note: auth.log, boot, btmp, daemon.log, debug, dmesg, kern.log, mail.info, mail.log, mail.warn, messages, syslog, udev, wtmp
</pre>

**if command are limited , you break out the &#8220;jail&#8221; shell**

<pre>python -c 'import pty;pty.spawn("/bin/bash")'
echo os.system('/bin/bash')
/bin/sh -i
</pre>

**How are system mounted**

<pre>mount
df -h
</pre>

 **are they any unmounted file-system**

<pre>cat /etc/fstab
</pre>

**What “Advanced Linux File Permissions” are used? Sticky bits, SUID & GUID**

<pre>find / -perm -1000 -type d 2&gt;/dev/null   # Sticky bit - Only the owner of the directory or the owner of a file can delete or rename here.
find / -perm -g=s -type f 2&gt;/dev/null    # SGID (chmod 2000) - run as the group, not the user who started it.
find / -perm -u=s -type f 2&gt;/dev/null    # SUID (chmod 4000) - run as the owner, not the user who started it.

find / -perm -g=s -o -perm -u=s -type f 2&gt;/dev/null    # SGID or SUID
for i in `locate -r "bin$"`; do find $i \( -perm -4000 -o -perm -2000 \) -type f 2&gt;/dev/null; done    # Looks in 'common' places: /bin, /sbin, /usr/bin, /usr/sbin, /usr/local/bin, /usr/local/sbin and any other *bin, for SGID or SUID (Quicker search)

# find starting at root (/), SGID or SUID, not Symbolic links, only 3 folders deep, list with more detail and hide any errors (e.g. permission denied)
find / -perm -g=s -o -perm -4000 ! -type l -maxdepth 3 -exec ls -ld {} \; 2&gt;/dev/null
</pre>

**Where can written to and executed from? A few ‘common’ places: /tmp, /var/tmp, /dev/shm**

<pre>find / -writable -type d 2&gt;/dev/null      # world-writeable folders
find / -perm -222 -type d 2&gt;/dev/null     # world-writeable folders
find / -perm -o w -type d 2&gt;/dev/null     # world-writeable folders

find / -perm -o x -type d 2&gt;/dev/null     # world-executable folders

find / \( -perm -o w -perm -o x \) -type d 2&gt;/dev/null   # world-writeable & executable folders

</pre>

**Any “problem” files? Word-writeable, “nobody” files**

<pre>find / -xdev -type d \( -perm -0002 -a ! -perm -1000 \) -print   # world-writeable files
find /dir -xdev \( -nouser -o -nogroup \) -print   # Noowner files

</pre>

**What development tools/languages are installed/supported?** 

<pre>find / -name perl*
find / -name python*
find / -name gcc*
find / -name cc
</pre>

**How can files be uploaded?**

<pre>find / -name wget
find / -name nc*
find / -name netcat*
find / -name tftp*
find / -name ftp
</pre>

**
  
Bash Script Untuk mendapatkan informasi basic**

<pre>#!/bin/sh
echo Distribution and kernel version
cat /etc/issue
uname -a

echo Mounted filesystems
mount -l

echo Network configuration
ifconfig -a
cat /etc/hosts
arp

echo Development tools availability
which gcc
which g++
which python

echo Installed packages (Ubuntu)
dpkg -l

echo Services
netstat -tulnpe

echo Processes
ps -aux

echo Scheduled jobs
find /etc/cron* -ls 2&gt;/dev/null
find /var/spool/cron* -ls 2&gt;/dev/null

echo Readable files in /etc 
find /etc -user `id -u` -perm -u=r \
 -o -group `id -g` -perm -g=r \
 -o -perm -o=r \
 -ls 2&gt;/dev/null 

echo SUID and GUID writable files
find / -o -group `id -g` -perm -g=w -perm -u=s \
 -o -perm -o=w -perm -u=s \
 -o -perm -o=w -perm -g=s \
 -ls 2&gt;/dev/null 

echo SUID and GUID files
find / -type f -perm -u=s -o -type f -perm -g=s \
 -ls 2&gt;/dev/null

echo Writable files outside HOME
mount -l find / -path “$HOME” -prune -o -path “/proc” -prune -o \( ! -type l \) \( -user `id -u` -perm -u=w  -o -group `id -g` -perm -g=w  -o -perm -o=w \) -ls 2&gt;/dev/null</pre>

&nbsp;

#### Finding exploit code

<http://www.exploit-db.com>

<http://1337day.com>

<http://www.securiteam.com>

<http://www.securityfocus.com>

<http://www.exploitsearch.net>

<http://metasploit.com/modules/>

<http://securityreason.com>

<http://seclists.org/fulldisclosure/>

<http://www.google.com>

#### Finding more information regarding the exploit

<http://www.cvedetails.com>

`http://packetstormsecurity.org/files/cve/[CVE]`

`http://cve.mitre.org/cgi-bin/cvename.cgi?name=[CVE]`

`http://www.vulnview.com/cve-details.php?cvename=[CVE]`

#### (Quick) “Common” exploits. _Warning. Pre-compiled binaries files. Use at your own risk_

<http://web.archive.org/web/20111118031158/http://tarantula.by.ru/localroot/>

<http://www.kecepatan.66ghz.com/file/local-root-exploit-priv9/>

## Mitigations

#### Is any of the above information easy to find?

Try doing it! Setup a cron job which automates script(s) and/or 3rd party products

#### Is the system fully patched?

_Kernel, operating system, all applications, their plugins and web services_<figure class="code"> <figcaption></figcaption> 

<div class="highlight">
  <table>
    <tr>
      <td class="gutter">
        <pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span></pre>
      </td>
      
      <td class="code">
        <pre><code class="bash">&lt;span class="line">apt-get update &lt;span class="o">&&&lt;/span> apt-get upgrade
&lt;/span>&lt;span class="line">yum update
&lt;/span></code></pre>
      </td>
    </tr>
  </table>
</div></figure> 

#### Are services running with the minimum level of privileges required?

For example, do you need to run MySQL as root?

#### Scripts _Can any of this be automated?!_

<http://pentestmonkey.net/tools/unix-privesc-check/>

<http://labs.portcullis.co.uk/application/enum4linux/>

<http://bastille-linux.sourceforge.net>

## Other (quick) guides & Links

#### Enumeration

<http://www.0daysecurity.com/penetration-testing/enumeration.html>

<http://www.microloft.co.uk/hacking/hacking3.htm>

#### Misc

<http://jon.oberheide.org/files/stackjacking-infiltrate11.pdf>

[http://pentest.cryptocity.net/files/operations/2009/post\_exploitation\_fall09.pdf](http://pentest.cryptocity.net/files/operations/2009/post_exploitation_fall09.pdf)

<http://insidetrust.blogspot.com/2011/04/quick-guide-to-linux-privilege.html>

Source : http://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/