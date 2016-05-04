---
layout: post
title: "Ways to do python post request"
description: "Well I'am usually using php to solve web challanges at ctf but python is also good to do that"
headline: 
modified: 2016-05-03
category: ctf
tags: [ctf,python]
imagefeature: /
mathjax: 
chart: 
comments: true
featured: true
---

Well I'am usually using php to solve web challanges at ctf but python is also good to do that.
since I'am new to learn python I facing some problem that weird because I didn't know what the detail of the library 
Nah , Now here we are talking about urllib,urllib2,mechanize,request 

We will take a look deeper in that library first urllib we talk based on the documentation on the python.
The urllib module has been split into **parts** and **renamed** in Python 3 to urllib.request, urllib.parse, and urllib.error. Here is the thing to remember Urllib exsist in python3 but in a separate parts.

---

#### 1. Urlopen

`urllib.urlopen(url[, data[, proxies[, context]]])`

you can input 4 parameter on Urlopen but the 3 other is optional use that in situational condition. the most important thing is the url where do want to open. 

After you call the Urlopen method you can use the method on it : *read(), [readline()](https://docs.python.org/2/library/readline.html#module-readline), readlines(), fileno(), close(), info(), getcode() and geturl()*


**geturl()** : returns the real URL of the page. <br> 
**getcode()** : method returns the HTTP status code <br>
**info()** : return an instance of HTTPMessage, to find all the method you can use inspect. <br>
**read()** : return an str content of the website opened <br>


```python

import urllib
import inspect

a = urllib.urlopen("https://ctfs.me/")
print(type(a))
print("Read() return : ")
print(type(a.read()))
print("geturl() return : ", a.geturl())
print(type(a.geturl()))
print("getcode() return : ", a.getcode())
print(type(a.getcode()))
print("close() return : ", a.close())
print(type(a.close()))
print("info() return : " , a.info().values())
print(type(a.info()))

# Debug method for a.info() Instances
#print(inspect.getmembers(a.info(), predicate=inspect.ismethod))

```

#### 2. Urlretrieve 

`urllib.urlretrieve(url[, filename[, reporthook[, data]]])`

you also can input 4 parameter on Urlretrieve


#### 3. Urlopener

`urllib._urlopener`

#### 4. Urlcleanup

` urllib.urlcleanup()`


### Utility functions

