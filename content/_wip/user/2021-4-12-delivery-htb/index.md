---
layout: post
title: HackTheBox Write-up &mdash; Delivery
---

This machine has IP 10.10.10.222.

We start off by enumerating the ports:
```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-12 16:15 NZST
Nmap scan report for delivery (10.10.10.222)
Host is up (0.25s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 9c:40:fa:85:9b:01:ac:ac:0e:bc:0c:19:51:8a:ee:27 (RSA)
|   256 5a:0c:c0:3b:9b:76:55:2e:6e:c4:f4:b9:5d:76:17:09 (ECDSA)
|_  256 b7:9d:f7:48:9d:a2:f2:76:30:fd:42:d3:35:3a:80:8c (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Welcome
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 53.57 seconds
```

We go to the website and it seems this is an email-related support group.

Following the prompts, it seems we need to go to `helpdesk.delivery.htb` to get an account.  From there, it seems we can go to `delivery.htb` on port `8065` to see a MatterMost server.

Looking in the source code of the main page, I see some tables, but I think I have to follow some prompts to actually see these tables.  It looks like they contain filler/blind text.

We go to `helpdesk.deliver.htb` to make an account.  I make an account using a fake email, but it sends a verification email.  So it does not check that email is valid, I guess.  I create a new ticket and it allows arbitrary file uploads, but I need to verify my account in order to view the ticket (6453400)...

Going back to the MatterMost server on `8065`, I am curious.  I search MatterMost:
> MatterMost is an open-source, self-hostable online chat service with file sharing, search, and integrations. It is designed as an internal chat for organisations and companies, and mostly markets itself as an open-source alternative to Slack and Microsoft Teams.

Okay, well it seems quite new, and it's written in Go.  Can't find anything on it in exploitDB (metasploit), so probably not much to do here.  However, it does look like we can create an account.  I try, but it requires verification, just like the helpdesk.  I try with a throwaway email, but nothing is sent.

In the "Contact Us" section of the main page, it says
> For unregistered users, please use our HelpDesk to get in touch with our team. Once you have an @delivery.htb email address, you’ll be able to have access to our MatterMost server.

So we know we need a `@delivery.htb` email to submit a support ticket.  I go to make a ticket (with an example email address: `ct@example.com`) and once it is submitted, I get am email address with the ticket number: `1839277@delivery.htb`.  Now we can go back to the MatterMost server and make an account using out `delivery.htb` email.  Now the confirmation email will sent do the support ticket email!  Indeed, we go to "View Ticket Thread", and put in my `ct@example.com` email, and ticket number, and we have main.  We click on the verification link, and put in the password we chose, and we are logged into MatterMost.

Now we see a chat:

![_config.yml]({{ site.baseurl }}/images/delivery-mattermost-chat.png)

We note that RockYou (mentioned above) is a common wordlist of passwords.  It looks like, mentioned here, are some `ssh` credentials.  We recall that an `ssh` port was open on this machine, so let's give it a go:
```bash
┌─[jakeireland@jake-mbp2017: pentesting]
└──╼ $ ssh maildeliverer@10.10.10.222
The authenticity of host '10.10.10.222 (10.10.10.222)' can't be established.
ECDSA key fingerprint is SHA256:LKngIDlEjP2k8M7IAUkAoFgY/MbVVbMqvrFA6CUrHoM.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.10.10.222' (ECDSA) to the list of known hosts.
maildeliverer@10.10.10.222's password:
Linux Delivery 4.19.0-13-amd64 #1 SMP Debian 4.19.160-2 (2020-11-28) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon Apr 12 17:09:36 2021 from 10.10.14.144
maildeliverer@Delivery:~$
```

Okay, so we have `ssh` access.  We get the user flag:
```bash
maildeliverer@Delivery:~$ whoami
maildeliverer
maildeliverer@Delivery:~$ ls /home
maildeliverer
maildeliverer@Delivery:~$ cat ~/user.txt
657630465d71e157475442fc95acab8d
```

https://drt.sh/posts/htb-delivery/#server-enumeration-and-privilege-escalation