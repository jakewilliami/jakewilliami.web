]# Passage

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


