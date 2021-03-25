---
layout: post
title: HackTheBox Write-up &mdash; Oopsie
---

I enumerate the box (which is apparently easy) and get the following:
```
$ bash common/nmap.sh oopsie     
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-22 20:37 NZDT
Nmap scan report for oopsie (10.10.10.28)
Host is up (0.24s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:e4:3f:d4:1e:e2:b2:f1:0d:3c:ed:36:28:36:67:c7 (RSA)
|   256 24:1d:a4:17:d4:e3:2a:9c:90:5c:30:58:8f:60:77:8d (ECDSA)
|_  256 78:03:0e:b4:a1:af:e5:c2:f9:8d:29:05:3e:29:c9:f2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Welcome
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.53 seconds
```

I go to `http://oopsie:80/` and we see a car-related website with the words `Bringing EV Ecosystem | Robust Design`.  I look at source and see it has an `images` subdomain that I don't have access to.  I can enumerate subdomains using `gobuster` and we find:
```
$ bash common/gobuster.sh 'http://oopsie:80/'
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://oopsie:80/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Expanded:                true
[+] Timeout:                 10s
===============================================================
2021/03/22 20:44:31 Starting gobuster in directory enumeration mode
===============================================================
http://oopsie:80/.htaccess            (Status: 403) [Size: 271]
http://oopsie:80/.htpasswd            (Status: 403) [Size: 271]
http://oopsie:80/css                  (Status: 301) [Size: 298] [--> http://oopsie/css/]
http://oopsie:80/fonts                (Status: 301) [Size: 300] [--> http://oopsie/fonts/]
http://oopsie:80/images               (Status: 301) [Size: 301] [--> http://oopsie/images/]
http://oopsie:80/js                   (Status: 301) [Size: 297] [--> http://oopsie/js/]    
http://oopsie:80/server-status        (Status: 403) [Size: 271]                            
http://oopsie:80/themes               (Status: 301) [Size: 301] [--> http://oopsie/themes/]
http://oopsie:80/uploads              (Status: 301) [Size: 302] [--> http://oopsie/uploads/]
                                                                                            
===============================================================
2021/03/22 20:46:16 Finished
===============================================================
```
But we don't have access to any of these.

Throughout the page's source code, there are references to Hamburgers for some reason.  Noted.  Also in the source code I find `http://oopsie/cdn-cgi/login/script.js`, so I go to the subdomain `http://oopsie/cdn-cgi/login/` and I get to a login page.  This login page redirects to `http://oopsie/cdn-cgi/login/index.php`.  My immediate thought is to use SQL injections, because this is what a simple box would probably entail?

We look at the `index.php` code here:
```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="apple-mobile-web-app-title" content="CodePen">
<title>Login</title>
<link href='/css/1.css' rel='stylesheet' type='text/css'>
<link rel="stylesheet" href="/css/font-awesome.min.css">
<link rel="stylesheet" href="/css/normalize.min.css">
<style>
body {
  font-family: "Open Sans", sans-serif;
  height: 100vh;
  background: url("/images/1.jpg") 50% fixed;
  background-size: cover;
}

@keyframes spinner {
  0% {
    transform: rotateZ(0deg);
  }
  100% {
    transform: rotateZ(359deg);
  }
}
* {
  box-sizing: border-box;
}

.wrapper {
  display: flex;
  align-items: center;
  flex-direction: column;
  justify-content: center;
  width: 100%;
  min-height: 100%;
  padding: 20px;
  background: rgba(4, 40, 68, 0.85);
}

.login {
  border-radius: 2px 2px 5px 5px;
  padding: 10px 20px 20px 20px;
  width: 90%;
  max-width: 320px;
  background: #ffffff;
  position: relative;
  padding-bottom: 80px;
  box-shadow: 0px 1px 5px rgba(0, 0, 0, 0.3);
}
.login.loading button {
  max-height: 100%;
  padding-top: 50px;
}
.login.loading button .spinner {
  opacity: 1;
  top: 40%;
}
.login.ok button {
  background-color: #8bc34a;
}
.login.ok button .spinner {
  border-radius: 0;
  border-top-color: transparent;
  border-right-color: transparent;
  height: 20px;
  animation: none;
  transform: rotateZ(-45deg);
}
.login input {
  display: block;
  padding: 15px 10px;
  margin-bottom: 10px;
  width: 100%;
  border: 1px solid #ddd;
  transition: border-width 0.2s ease;
  border-radius: 2px;
  color: #ccc;
}
.login input + i.fa {
  color: #fff;
  font-size: 1em;
  position: absolute;
  margin-top: -47px;
  opacity: 0;
  left: 0;
  transition: all 0.1s ease-in;
}
.login input:focus {
  outline: none;
  color: #444;
  border-color: #2196F3;
  border-left-width: 35px;
}
.login input:focus + i.fa {
  opacity: 1;
  left: 30px;
  transition: all 0.25s ease-out;
}
.login a {
  font-size: 0.8em;
  color: #2196F3;
  text-decoration: none;
}
.login .title {
  color: #444;
  font-size: 1.2em;
  font-weight: bold;
  margin: 10px 0 30px 0;
  border-bottom: 1px solid #eee;
  padding-bottom: 20px;
}
.login button {
  width: 100%;
  height: 100%;
  padding: 10px 10px;
  background: #2196F3;
  color: #fff;
  display: block;
  border: none;
  margin-top: 20px;
  position: absolute;
  left: 0;
  bottom: 0;
  max-height: 60px;
  border: 0px solid rgba(0, 0, 0, 0.1);
  border-radius: 0 0 2px 2px;
  transform: rotateZ(0deg);
  transition: all 0.1s ease-out;
  border-bottom-width: 7px;
}
.login button .spinner {
  display: block;
  width: 40px;
  height: 40px;
  position: absolute;
  border: 4px solid #ffffff;
  border-top-color: rgba(255, 255, 255, 0.3);
  border-radius: 100%;
  left: 50%;
  top: 0;
  opacity: 0;
  margin-left: -20px;
  margin-top: -20px;
  animation: spinner 0.6s infinite linear;
  transition: top 0.3s 0.3s ease, opacity 0.3s 0.3s ease, border-radius 0.3s ease;
  box-shadow: 0px 1px 0px rgba(0, 0, 0, 0.2);
}
.login:not(.loading) button:hover {
  box-shadow: 0px 1px 3px #2196F3;
}
.login:not(.loading) button:focus {
  border-bottom-width: 4px;
}

footer {
  display: block;
  padding-top: 50px;
  text-align: center;
  color: #ddd;
  font-weight: normal;
  text-shadow: 0px -1px 0px rgba(0, 0, 0, 0.2);
  font-size: 0.8em;
}
footer a, footer a:link {
  color: #fff;
  text-decoration: none;
}
</style>
<script>
  window.console = window.console || function(t) {};
</script>
<script src="/js/prefixfree.min.js"></script>
<script>
  if (document.location.search.match(/type=embed/gi)) {
    window.parent.postMessage("resize", "*");
  }
</script>
</head>
<body translate="no">
<div class="wrapper">
<form class="login" action="/cdn-cgi/login/index.php" method="POST">
<p class="title">Log in</p>
<input type="text" placeholder="Username" name="username" autofocus />
<i class="fa fa-user"></i>
<input type="password" name="password" placeholder="Password" />
<i class="fa fa-key"></i>
<button>
<i class="spinner"></i>
<span class="state">Log in</span>
</button>
</form>
</p>
</div>
<script src="/js/min.js"></script>
<script id="rendered-js">
var working = false;
$('.login').on('submit', function (e) {
  e.preventDefault();
  if (working) return;
  working = true;
  var $this = $(this),
  $state = $this.find('button > .state');
  $this.addClass('loading');
  $state.html('Authenticating');
  setTimeout(function () {
    $this.addClass('ok');
    $state.html('Welcome back!');
    setTimeout(function () {
      $state.html('Log in');
      $this.removeClass('ok loading');
      working = false;
    }, 4000);
  }, 3000);
});
//# sourceURL=pen.js
    </script>
</body>
</html>
```

So we see that there is a `pen.js` subdomain/file after `index.php`.  

While searching for ways to penetrate using `pen.js`, I found [this seemingly useful resource](https://www.exploit-db.com/papers/12871).  