---
layout: post
title: HackTheBox Write-up &mdash; Laboratory
---

This machine has IP 10.10.10.216.

We enumerate its ports:

```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-12 14:52 NZST
Nmap scan report for 10.10.10.216
Host is up (0.27s latency).
Not shown: 997 filtered ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 25:ba:64:8f:79:9d:5d:95:97:2c:1b:b2:5e:9b:55:0d (RSA)
|   256 28:00:89:05:55:f9:a2:ea:3c:7d:70:ea:4d:ea:60:0f (ECDSA)
|_  256 77:20:ff:e9:46:c0:68:92:1a:0b:21:29:d1:53:aa:87 (ED25519)
80/tcp  open  http     Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to https://laboratory.htb/
443/tcp open  ssl/http Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: The Laboratory
| ssl-cert: Subject: commonName=laboratory.htb
| Subject Alternative Name: DNS:git.laboratory.htb
| Not valid before: 2020-07-05T10:39:28
|_Not valid after:  2024-03-03T10:39:28
| tls-alpn:
|_  http/1.1
Service Info: Host: laboratory.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 45.18 seconds
```

Note that there seems to be a git server associated with this!  

We go to the HTTP server and are met with a Security and Development Services website.  This is ironic!  And there is a little funny note at the bottom of the page:
```
RESPONSIBLE DISCLOSURE
Find a vulnerability in this website? You're lying! We code 100% secure, and I'm sure you can't hack us. If you do, definitely don't let us know.
```

I look at the source, and the header has an MP4 file: `http://laboratory/images/banner.mp4`, but it doesn't seem to load anything.  I download it and it displays someone scrolling through what looks like a Jekyll deploy log.  I try `http://laboratory/images/` and we get the following list of files (an open directory&mdash; yay):
```
banner.jpg
banner.mp4
bg.jpg
cta01.jpg
pic01.jpg
pic02.jpg
pic03.jpg
```

These are all images that are displayed on the main page.  I use `exiftool` and `identify -verbose` to see if there is anything important in the image files, but nothing that I can find (other than the computer model that made `pic01.jpg` using photoshop on Windows, but that doesn't necessarily pertain to the server).

I use `ffuf` to see if there are any other open directories:
```bash
.htpasswd               [Status: 403, Size: 276, Words: 20, Lines: 10]
.htaccess               [Status: 403, Size: 276, Words: 20, Lines: 10]
assets                  [Status: 301, Size: 311, Words: 20, Lines: 10]
images                  [Status: 301, Size: 311, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 276, Words: 20, Lines: 10]
```

So not much here.  We see that there is an `assets` subdomain, which shows us some of the code used in the webpage.  We also see this referenced in the scripts section at the bottom of the main page.

Let's investigate the git server.  Going to `https://git.laboratory.htb/` takes us to `http://git.laboratory.htb/users/sign_in`, which looks to be a self-hosted GitLab server.  Unfortunately, trying to make an account it says `Email domain is not authorized for sign-up`.

Enumerating the git repo, we find that the one thing which doesn't redirect us to `/users/sign_in` is a `robot.txt` file.  This gives us some directories that could not be picked up by `ffuf` or `gobuster`:
```
# Disallow: /
Disallow: /autocomplete/users
Disallow: /search
Disallow: /api
Disallow: /admin
Disallow: /profile
Disallow: /dashboard
Disallow: /projects/new
Disallow: /groups/new
Disallow: /users
Disallow: /help
Disallow: /s/
```

Looking through these, a lot of them gives us redirects back to `/users/sign_in`.  However, `/search` allows us to search for repositories, `/autocomplete/users` gives us a file with `[]` in it, and `help` gives us some help about this version of GitLab Community Edition.  This is a pretty standard help page for GitLab.

I search for `GitLab exploit` online, and find [this](https://www.rapid7.com/db/modules/exploit/multi/http/gitlab_file_read_rce/).