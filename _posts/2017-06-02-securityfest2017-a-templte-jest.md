---
private: 'false'
layout: post
featured: false
published: true
comments: true
title: SecurityFest2017 - A Temple Jest
tags: schemainjection
mathjax: false
imagefeature: /images/express.jpg
description: SecurityFest2017 - A Temple Jest
category: Web Exploitation
chart: null
---


## Problems

```
The target are setting up their intranet communication service. Hack it before they are done!
Solves: 52
Service: http://alieni.se:3003/
Author: avlidienbrunn
```


## Writeup

This web app using node and expressjs framework too we can read `package.json`


```js
{
  name: "atemplejest",
  version: "1.0.0",
  description: "A temple jest project",
  main: "index.js",
  scripts: {
    test: "echo "Error: no test specified" && exit 1"
  },
  repository: {
    type: "git",
    url: "git+https://github.com/avlidienbrunn/atemplejest.git"
  },
  keywords: [
  "asd"
  ],
  author: "avlidienbrunn",
  license: "WTFPL",
  bugs: {
    url: "https://github.com/avlidienbrunn/atemplejest/issues"
  },
  homepage: "https://github.com/avlidienbrunn/atemplejest#readme",
  dependencies: {
    ejs: "^2.5.6",
    threads: "^0.7.3"
  }
}
```

Trying to get the github repo without luck, because the repo was removed,
after tring some test `http://alieni.se:3003/render/<var>` i think we input in the parameter is variable then got eval'ed
we can do some aritmathic calculation like `5+6` then we can call `procces.env` => `[object Object] is under construction...`

we can leak memory using `Buffer()` to get the flag `https://snyk.io/blog/exploiting-buffer/`
executing this `http://alieni.se:3003/render/Buffer(100000)` then simply search the flag by hand.