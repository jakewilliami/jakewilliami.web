---
layout: post
title: HackTheBox Write-up &mdash; ScriptKiddie
---

This machine has IP 10.10.10.226.

Enumerating the box, we find:
```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-12 16:43 NZST
Nmap scan report for scriptkiddie (10.10.10.226)
Host is up (0.25s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 3c:65:6b:c2:df:b9:9d:62:74:27:a7:b8:a9:d3:25:2c (RSA)
|   256 b9:a1:78:5d:3c:1b:25:e0:3c:ef:67:8d:71:d3:a3:ec (ECDSA)
|_  256 8b:cf:41:82:c6:ac:ef:91:80:37:7c:c9:45:11:e8:43 (ED25519)
5000/tcp open  http    Werkzeug httpd 0.16.1 (Python 3.8.5)
|_http-server-header: Werkzeug/0.16.1 Python/3.8.5
|_http-title: k1d'5 h4ck3r t00l5
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.57 seconds
```

So we go to `http://scriptkiddie:5000/` and we find a website that allows us to run scripts to help us hacking.  

Let's intercept with Burp.  These are the body parameters to the `nmap` section:
```
ip=10.10.10.226&action=scan
```

I tried changing this to
```
ip=10.10.10.226;ls&action=scan
```

But it checks the validity of the IP address first.  

For the `sploits` section, `action=searchsploit`.  