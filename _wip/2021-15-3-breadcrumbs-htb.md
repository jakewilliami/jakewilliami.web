---
layout: post
title: HackTheBox Write-up &mdash; Breadcrumbs
---

This machine is a *Windows* machine on IP `10.10.10.228`.  This will be a challenge for it is a Windows machine, and my first one at that!

I see what ports I have access to!

```bash
/: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1h PHP/8.0.1
|_http-title: Library
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp  open  ssl/http      Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1h PHP/8.0.1)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1h PHP/8.0.1
|_http-title: Library
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds?
3306/tcp open  mysql?
| fingerprint-strings: 
|   JavaRMI, LANDesk-RC, NULL, RTSPRequest, TLSSessionReq: 
|_    Host '10.10.14.239' is not allowed to connect to this MariaDB server
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.91%I=7%D=3/14%Time=604EC478%P=x86_64-pc-linux-gnu%r(NU
SF:LL,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.239'\x20is\x20not\x20allo
SF:wed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(RTSPRequest
SF:,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.239'\x20is\x20not\x20allowe
SF:d\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(TLSSessionReq
SF:,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.239'\x20is\x20not\x20allowe
SF:d\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(LANDesk-RC,4B
SF:,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.239'\x20is\x20not\x20allowed\x
SF:20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(JavaRMI,4B,"G\0\
SF:0\x01\xffj\x04Host\x20'10\.10\.14\.239'\x20is\x20not\x20allowed\x20to\x
SF:20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -56m50s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-03-15T01:24:13
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 75.32 seconds
```

Notice a few different terms here:
  - `msrpc`
  - `netbios-ssn`

I also search for `msrpc` ([see here](https://www.extrahop.com/resources/protocols/msrpc/)).  This pertains to *Windows RPC*, which seems to stand for "Remote Procedure Call" (also known as a function call or a subtroutine call); a protocol that uses the client0server model in order to allow one programme to request service from a programme or another computer without having to understand the details of that computer's network.

Searching `netbios-ssn` [we find](https://www.wikiwand.com/en/NetBIOS) we find it is a tool providing services allowing applications on separate computers to communicate over a LAN.

We also see an `http` port open on `443`.  Let's add this machine to our hosts file for ease of reference:
```bash
echo "10.10.10.228" breadcrumbs | sudo tee -a /etc/hosts
```

Now we go onto 
```
http://breadcrumbs:443/
```

And we get this message:
```
Bad Request

Your browser sent a request that this server could not understand.
Reason: You're speaking plain HTTP to an SSL-enabled server port.
Instead use the HTTPS scheme to access this URL, please.
```

[We know](https://github.com/jakewilliami/http-status-cheat-sheet/blob/master/http-status-cheat-sheet.pdf) that `HTTP` status code `400` (Bad Request) means that the server cannot or will not process the request due to an apparent client error.

We try again, this time without a port to let the server do the work:
```
http://breadcrumbs/
```
And we find this online library website.  Interesting.  A quick look at the source code reveals an open directory!: `http://breadcrumbs/php/books.php led me to 
```
http://breadcrumbs/php/
```

But in fact, this open directory has only one element in it, and that is the `books.php` site which we already have access to.  This is an interface where you can enter a title and an author, and search for books.  I enter the book I am currently reading: Crime and Punishment, by Fyodor Dostoyevsky.  But we get the message: `Nothing found :(`.

What my immediate thought is, is SQL Injection.  It is likely searching from a database, and we see `mysql` on port `3306`, so let's give it a go!

The search query should look something like this:
```sql
SELECT ? FROM ? WHERE ? LIKE '%SEARCHQUERY%';
```

We want to escape the search query and then run something.

I type `%` into either search bar, and it comes up with all of the things in the database.  This is good, as `%` is a wildcard character.

I also tried typing
```
';--
```
which should have also come up with everything, but that returned nothing.  We also note that each book has a description, and a max borrow duration, so the database may have four or five columns.

This tells us that one of the following is what we have:
```sql
SELECT ? FROM ? WHERE Title LIKE '%' OR Author LIKE '%';
SELECT ? FROM ? WHERE Title LIKE "%" OR Author LIKE "%";
SELECT ? FROM ? WHERE Title LIKE % OR Author LIKE %;
```

We can try putting a comment in early:
```sql
SELECT ? FROM ? WHERE Title LIKE "%";-- " OR Author LIKE "%";
```

What I am trying to do is stop the SQL Query before author, and put `Twain` as an author.  I think when I do this, this would mean that Twain does not appear.

So I try putting in `';--` in the Title part, but Tom Sawyer still shows up...

But I think I have been going about this wrong.  This is what may be actually happening:
```sql
inputTitle = getRequestString("Sawyer");
inputAuthor = getRequestString("Twain");
tSQL = "SELECT ? FROM ? WHERE Title = " + inputTitle + "OR WHERE Author = " + inputAuthor;
```

The fact that putting `'` into the search bar comes up with `Alice's Adventures in Wonderland` makes me think that the single quote is escaped correctly, unfortunately.

I have spent a long time trying to get this to work.  I think there is *something* here, because I can use the SQL wildcard, but I think I need to have another look at the source code of the page for more information.

I just found `view-source:http://breadcrumbs/js/books.js` which contains the following code:
```javascript
$(document).ready(function(){
    var book = null;
    $("#note").click(function(){
        $("#tableBody").html("");
        const title = $("#title").val();
        const author = $("#author").val();
        if(title == "" && author == ""){
            $("#message").html("Nothing found :(");
        }
        else{
            searchBooks(title, author);
        }
    })

    $("#interested").click(function(){

    });
});

function getInfo(e){
    const bookId = "book" + $(e).closest('tr').attr('id') + ".html";
    jQuery.ajax({
        url: "../includes/bookController.php",
        type: "POST",
        data: {
            book: bookId,
            method: 1,
        },
        dataType: "json",
        success: function(res){
            $("#about").html(res);
        }
    });
}

function modal(){
    return '<button type="button" onclick="getInfo(this)" class="btn btn-outline-warning" data-toggle="modal" data-target="#actionModal">Book</button>';
}

function searchBooks(title, author){
    jQuery.ajax({
        url: "../includes/bookController.php",
        type: "POST",
        data: {
            title: title,
            author: author,
            method: 0,
        },
        dataType: "json",
        success: function(res){
            if(res.length == 0 || res == false){
                $("#message").html("Nothing found :(");
            }
            else{
                let ret = "";
                for(book in res){
                    $("#message").html("");
                    ret += "<tr id='" + res[book].id + "'>";
                    ret += "<td>"+res[book].title+"</td>";
                    ret += "<td>"+res[book].author+"</td>";
                    ret += "<td>" + modal() + "</td>";
                    ret += "</tr>";
                    $("#tableBody").html(ret)
                }
            }
        }
    });
}
```

So perhaps I had gone the wrong path with SQL Injections.  It looks like it uses an ajax query.  I think that perhaps one important bit lies here:
```javascript
let ret = "";
for(book in res){
	$("#message").html("");
	ret += "<tr id='" + res[book].id + "'>";
	ret += "<td>"+res[book].title+"</td>";
	ret += "<td>"+res[book].author+"</td>";
	ret += "<td>" + modal() + "</td>";
	ret += "</tr>";
	$("#tableBody").html(ret)
}
```

This programmatically produces a table in HTML, but gives us some valuable insight into how the table data is called.

What this does tell us is that there are at least three (possibly four) columns in the database: `id`, `title`, `author`, and possibly information.

Going into `http://breadcrumbs/js/` we see a `main.js` at `http://breadcrumbs/js/main.js`, but we cannot view this for some reason.  Though, it returns status code `200`, so perhaps it is an empty file?

We can also go to `http://breadcrumbs/includes/` and see `http://breadcrumbs/includes/bookController.php`, which is apparently empty as well.

Let's enumerate the page to see if there is any other ones we can find, other than `includes` and `js`:
```bash
$ gobuster dir -w /usr/share/wordlists/dirb/big.txt -t 50 -e -u http://breadcrumbs/
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://breadcrumbs/
[+] Threads:        50
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Expanded:       true
[+] Timeout:        10s
===============================================================
2021/03/15 22:18:40 Starting gobuster
===============================================================
http://breadcrumbs/.htaccess (Status: 403)
http://breadcrumbs/.htpasswd (Status: 403)
http://breadcrumbs/Books (Status: 301)
http://breadcrumbs/DB (Status: 301)
http://breadcrumbs/PHP (Status: 301)
http://breadcrumbs/aux (Status: 403)
http://breadcrumbs/books (Status: 301)
http://breadcrumbs/cgi-bin/ (Status: 403)
http://breadcrumbs/com1 (Status: 403)
http://breadcrumbs/com2 (Status: 403)
http://breadcrumbs/com4 (Status: 403)
http://breadcrumbs/com3 (Status: 403)
http://breadcrumbs/con (Status: 403)
http://breadcrumbs/css (Status: 301)
http://breadcrumbs/db (Status: 301)
http://breadcrumbs/includes (Status: 301)
http://breadcrumbs/js (Status: 301)
http://breadcrumbs/licenses (Status: 403)
http://breadcrumbs/lpt2 (Status: 403)
http://breadcrumbs/lpt1 (Status: 403)
http://breadcrumbs/nul (Status: 403)
http://breadcrumbs/php (Status: 301)
http://breadcrumbs/phpmyadmin (Status: 403)
http://breadcrumbs/portal (Status: 301)
http://breadcrumbs/prn (Status: 403)
http://breadcrumbs/secci� (Status: 403)
http://breadcrumbs/server-info (Status: 403)
http://breadcrumbs/server-status (Status: 403)
http://breadcrumbs/webalizer (Status: 403)
===============================================================
2021/03/15 22:20:32 Finished
===============================================================
```

I go to `http://breadcrumbs/books/` and find an open directory of HTML files!  This is where the information of each book is kept, it seems.

Also going to `http://breadcrumbs/DB/` we find a file, `db.php`, but it seems empty.

Going to `http://breadcrumbs/portal/` we see a login, and a message saying
```
Restricted domain for: 10.10.14.239
Please return home or contact helper if you think there is a mistake.
```

If you click on `helper` you get to the webpage `http://breadcrumbs/portal/php/admins.php`.  This page shows a table This means they surely have some database of users we can access!

Going back to the login portal, we can go to a signup link.  We sign up and log in, and now we have access to the following site:
![_config.yml]({{ site.baseurl }}/images/breadcrumbs-portal-binary.png)

Now we have access to this site: `http://breadcrumbs/portal/php/users.php`.  This gives us all of the users' names, ages, and positions:

<table align="center">
	<thead>
		<tr>
			<td>Username</td>
			<td>Age</td>
			<td>Position</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><code>alex</code></td>
			<td>21</td>
			<td>Admin</td>
		</tr>
		<tr>
			<td><code>paul</code></td>
			<td>24</td>
			<td>Admin</td>
		</tr>
		<tr>
			<td><code>jack</code></td>
			<td>22</td>
			<td>Admin</td>
		</tr>
		<tr>
			<td><code>olivia</code></td>
			<td>24</td>
			<td>Data Analyst</td>
		</tr>
		<tr>
			<td><code>john</code></td>
			<td>39</td>
			<td>Ad Manager</td>
		</tr>
		<tr>
			<td><code>emma</code></td>
			<td>20</td>
			<td>Developer</td>
		</tr>
		<tr>
			<td><code>william</code></td>
			<td>20</td>
			<td>Developer</td>
		</tr>
		<tr>
			<td><code>lucas</code></td>
			<td>25</td>
			<td>Developer</td>
		</tr>
		<tr>
			<td><code>sirine</code></td>
			<td>27</td>
			<td>Reception</td>
		</tr>
		<tr>
			<td><code>juliette</code></td>
			<td>20</td>
			<td>Server Admin</td>
		</tr>
		<tr>
			<td><code>support</code></td>
			<td>-</td>
			<td>Service</td>
		</tr>
		<tr>
			<td><code>christophertatlock</code></td>
			<td>-</td>
			<td>Awaiting approval</td>
		</tr>
	</tbody>
</table>

There is also an Issues tab, which has some issues:
![_config.yml]({{ site.baseurl }}/images/breadcrumbs-portal-issues.png)

The fact that this box is called "Breadcrumbs" seems to me that I should *follow the breadcrumbs*, as it were, and look for an exploit using that logout button...  It is also important to note the other issue: `Fix PHPSESSID infinite session duration`.  I think this could be useful.

Just a side note, but I think this is very funny:
![_config.yml]({{ site.baseurl }}/images/breadcrumbs-portal-pizza.png)

Finally, there is a File Management tab, but it always redirects me to my user home, presumably because I don't have access rights to files...

I access my session cookies:
<table align="center">
	<thead>
		<tr>
			<td>Name</td>
			<td>Value</td>
			<td>Domain</td>
			<td>Expired / Max-Age</td>
			<td>Size</td>
			<td>HttpOnly</td>
			<td>Secure</td>
			<td>SameSite</td>
			<td>Last Accessed</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><code>PHPSESSID</code></td>
			<td><code>christophertatlock09e0edc23e6382227ca9de420edb3ed0</code></td>
			<td>breadcrumbs</td>
			<td>Session</td>
			<td>59</td>
			<td><code>false</code></td>
			<td><code>false</code></td>
			<td>None</td>
			<td>Tue, 16 Mar 2021 07:20:03 GMT</td>
		</tr>
		<tr>
			<td>token</td>
			<td><code>eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjp7InVzZXJuYW1lIjoiY2hyaXN0b3BoZXJ0YXRsb2NrIn19.5JlFmYKQUpQ4VEtuRp77FxrmY0BJBZReewDJR0uDqFQ</code></td>
			<td>breadcrumbs</td>
			<td>Thu, 15 Apr 2021 07:20:03 GMT</td>
			<td>142</td>
			<td><code>false</code></td>
			<td><code>false</code></td>
			<td>None</td>
			<td>Tue, 16 Mar 2021 07:20:03 GMT</td>
		</tr>
	</tbody>
</table>

The important thing is the `PHPSESSID` code, which apparently last forever.  Surely we can use that for an exploit.