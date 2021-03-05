---
layout: post
title: HackTheBox Write-up &mdash; Bucket
---

This machine is a Linux machine on IP `10.10.10.212`.

Running an `nmap` scan, we see two ports open:
```bash
$ nmap -sC -sV 10.10.10.212
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-03 03:58 EST
Stats: 0:00:03 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 25.07% done; ETC: 03:58 (0:00:06 remaining)
Nmap scan report for 10.10.10.212
Host is up (0.24s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp open  http    Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://bucket.htb/
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
 
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 48.61 seconds
```

Just like Passage, we have an `ssh` and an `http` port open.  Let's have a look at the `http` port!

We will add the machine to our `hosts` file for ease of use:
```bash
echo "10.10.10.212 bucket" | sudo tee -a /etc/hosts
```

We go to `http://bucket:80`, which redirects us to `http://bucket.htb`, but we come to an error: `we can't connect to server at bucket.htb`, or `we're having trouble finding that site`.  Looking closer at the `nmap` output, we see that the `http-title` "Did not follow redirect".

Running a more comprehensive search or ports up to port number `65535` we find
```bash
$ sudo masscan -e tun0 -p1-65535 --rate=1000 10.10.10.212                                                         1 ⨯

Starting masscan 1.0.5 (http://bit.ly/14GZzcT) at 2021-03-03 09:17:45 GMT
 -- forced options: -sS -Pn -n --randomize-hosts -v --send-eth
Initiating SYN Stealth Scan
Scanning 1 hosts [65535 ports/host]
Discovered open port 80/tcp on 10.10.10.212                                    
Discovered open port 22/tcp on 10.10.10.212     
```
```bash
$ mkdir nmap && touch full.nmap && sudo nmap -sC -sV -O -p- -oA nmap/full 10.10.10.212 && cat full.nmap            1 ⨯
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-03 04:26 EST
Stats: 0:09:42 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 44.50% done; ETC: 04:48 (0:12:05 remaining)
Stats: 0:19:54 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 71.95% done; ETC: 04:54 (0:07:45 remaining)
Nmap scan report for bucket (10.10.10.212)
Host is up (0.23s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|    f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp open  http    Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://bucket.htb/
Aggressive OS guesses: Linux 4.15 - 5.6 (95%), Linux 5.3 - 5.4 (95%), Linux 2.6.32 (95%), Linux 5.0 - 5.3 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 5.0 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1841.64 seconds~
```

So this isn't very different.  Let's try UDP scan:
```bash
$ sudo nmap -sUV -T4 10.10.10.212
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-03 05:01 EST
Warning: 10.10.10.212 giving up on port because retransmission cap hit (6).
Stats: 0:20:42 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 6.25% done; ETC: 05:37 (0:15:45 remaining)
Nmap scan report for bucket (10.10.10.212)
Host is up (0.35s latency).
All 1000 scanned ports on bucket (10.10.10.212) are closed (968) or open|filtered (32)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1291.65 seconds

$ sudo nmap -sUV -F 10.10.10.212
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-03 05:23 EST
Nmap scan report for bucket (10.10.10.212)
Host is up (0.39s latency).
All 100 scanned ports on bucket (10.10.10.212) are closed

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 103.29 seconds

$ sudo nmap -sU -O -p- -oA nmap/udp 10.10.10.212
Nmap scan report for bucket (10.10.10.212)
Host is up (0.26s latency).
All 65535 scanned ports on bucket (10.10.10.212) are closed (65324) or open|filtered (211)
Too many fingerprints match this host to give specific OS details
Network Distance: 2 hops

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 85588.49 seconds
```
Nothing here... (but it took a minute)!

After some searching on Linux forums, it looks like the problem might in fact be in my `hosts` file.  Changing
```bash
10.10.10.212 bucket
```
to
```bash
10.10.10.212 bucket.htb
```
fixed this issue.  This took me annoyingly long to figure out, but good to know!

It looks like the page is an advertising platform.  There are three main links at the top: `Home`, `About`, and `Feed`.  As usual, let's look at the page source!

The only thing of interest to me in the page source is that the server pulls (or attempts to pull) images from the following website:
```
http://s3.bucket.htb/adserver/images/
```

Though I can't seem to get to this website (via `ping` nor browser).













After a bit of searching, I found a tool to check what subdomains there might be with a given site.  This tool is called `gobuster`.  

I first tried it with the initial site:
```bash
$ gobuster dir -w /usr/share/wordlists/dirb/big.txt -t 50 -e -u http://bucket.htb                               1 ⨯
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://bucket.htb
[+] Threads:        50
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Expanded:       true
[+] Timeout:        10s
===============================================================
2021/03/04 06:05:46 Starting gobuster
===============================================================
http://bucket.htb/.htpasswd (Status: 403)
http://bucket.htb/.htaccess (Status: 403)
http://bucket.htb/server-status (Status: 403)
===============================================================
2021/03/04 06:07:48 Finished
===============================================================
```

Then I tried it with the image site: