---
layout: post
title: HackTheBox Write-up &mdash; The Notebook
---

This machine is a Linux machine on IP `10.10.10.230`.

This is the result of running `nmap`:
```bash
$ bash common/nmap.sh 10.10.10.230
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-21 19:24 NZST
Nmap scan report for 10.10.10.230
Host is up (0.23s latency).
Not shown: 996 closed ports
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 86:df:10:fd:27:a3:fb:d8:36:a7:ed:90:95:33:f5:bf (RSA)
|   256 e7:81:d6:6c:df:ce:b7:30:03:91:5c:b5:13:42:06:44 (ECDSA)
|_  256 c6:06:34:c7:fc:00:c4:62:06:c2:36:0e:ee:5e:bf:6b (ED25519)
80/tcp    open     http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: The Notebook - Your Note Keeper
8000/tcp  open     http    SimpleHTTPServer 0.6 (Python 3.6.9)
|_http-server-header: SimpleHTTP/0.6 Python/3.6.9
|_http-title: Directory listing for /
10010/tcp filtered rxapi
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.99 seconds
```

So I go onto `10.10.10.230:80` and make an account.  I see, in Burp Suite, that all of my requests have the following `Cookie` request header:
```
Cookie: auth=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6NzA3MC9wcml2S2V5LmtleSJ9.eyJ1c2VybmFtZSI6ImNocmlzdG9waGVydGF0bG9jayIsImVtYWlsIjoiZXZpbEBnbWFpbC5jb20iLCJhZG1pbl9jYXAiOmZhbHNlfQ.URyro-frBO8JzwXAKVSHvFJUETP5I8HWPIADoq8i30eCxVz5j0lOXDH5UoffS0H5Hcerxvsh0cvAy_1l0E32AcVrS41FQWCQ8Gy0KPMessX6-2GER-r-_NbNRWh_IEVTQZULhS1AL3aMOeH0-LnmVe1xOkhRusQ2_5ALt1KYSWO_c8vgSLG0oSLiXiRwM3BFTrMIRbtKVtW2cV6hwt_cxCNzSupIlti0jvR1qt7vat1ChFEhUwOtZfdAUN6gX7Oa05IwVVzHgNjYLD9gMG7SFDQLPEWfsEmgXM31XIBYZq5KJjDwzOOAvExSYiHHyGGBwoHdjBYdgs56LFyJuiWuEeltiwqRceR-zKUa9Xzfk-I36STzIYnJ4Is1Qbv6DFDY_uYQrftW5UJiXKC9Crx7oG6kp2VKVVVM_13rV1cK9NoCK_5U5j6QIw_8uEG74dU6Xsx_aQ3cR_Y8i3SeGI1yb5Z6LarmeU1oNX7yT04h2dm5Lg2IhVGbc1loL6S1rXhRPwIEgyZSply0IyzKPf4rVeccooedNganP07nwEjDQN-imXxsA_nXmwR1Bj_81guiDLvEHAQN6pJBx5McCDxvjJzWUrngSNQj8SARKl5mk7YpNIpM8U__fXACxI-2VqwQKzQ89Sz1F3Wo-3Fijke972cuwdfw76gnJwKaadCwOYI; uuid=57fa1eda-c1c6-43c5-bf39-b0550c0af4bc
```
I notice this looks like a JWT, so I use the following script to decode it:
```julia
using JWTs

token = "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6NzA3MC9wcml2S2V5LmtleSJ9.eyJ1c2VybmFtZSI6ImNocmlzdG9waGVydGF0bG9jayIsImVtYWlsIjoiZXZpbEBnbWFpbC5jb20iLCJhZG1pbl9jYXAiOmZhbHNlfQ.URyro-frBO8JzwXAKVSHvFJUETP5I8HWPIADoq8i30eCxVz5j0lOXDH5UoffS0H5Hcerxvsh0cvAy_1l0E32AcVrS41FQWCQ8Gy0KPMessX6-2GER-r-_NbNRWh_IEVTQZULhS1AL3aMOeH0-LnmVe1xOkhRusQ2_5ALt1KYSWO_c8vgSLG0oSLiXiRwM3BFTrMIRbtKVtW2cV6hwt_cxCNzSupIlti0jvR1qt7vat1ChFEhUwOtZfdAUN6gX7Oa05IwVVzHgNjYLD9gMG7SFDQLPEWfsEmgXM31XIBYZq5KJjDwzOOAvExSYiHHyGGBwoHdjBYdgs56LFyJuiWuEeltiwqRceR-zKUa9Xzfk-I36STzIYnJ4Is1Qbv6DFDY_uYQrftW5UJiXKC9Crx7oG6kp2VKVVVM_13rV1cK9NoCK_5U5j6QIw_8uEG74dU6Xsx_aQ3cR_Y8i3SeGI1yb5Z6LarmeU1oNX7yT04h2dm5Lg2IhVGbc1loL6S1rXhRPwIEgyZSply0IyzKPf4rVeccooedNganP07nwEjDQN-imXxsA_nXmwR1Bj_81guiDLvEHAQN6pJBx5McCDxvjJzWUrngSNQj8SARKl5mk7YpNIpM8U__fXACxI-2VqwQKzQ89Sz1F3Wo-3Fijke972cuwdfw76gnJwKaadCwOYI"
header, payload, signature = String.(split(token, '.'))

for P in JWTs.decodepart.([header, payload])
    for (k, v) in P
        println("$k => $v")
    end
end

println(); println("Signature:\n$signature")
```

I get the following response:
```julia
julia> include("jwt.jl")
typ => JWT
alg => RS256
kid => http://localhost:7070/privKey.key
username => christophertatlock
email => evil@gmail.com
admin_cap => false

Verify signature:
URyro-frBO8JzwXAKVSHvFJUETP5I8HWPIADoq8i30eCxVz5j0lOXDH5UoffS0H5Hcerxvsh0cvAy_1l0E32AcVrS41FQWCQ8Gy0KPMessX6-2GER-r-_NbNRWh_IEVTQZULhS1AL3aMOeH0-LnmVe1xOkhRusQ2_5ALt1KYSWO_c8vgSLG0oSLiXiRwM3BFTrMIRbtKVtW2cV6hwt_cxCNzSupIlti0jvR1qt7vat1ChFEhUwOtZfdAUN6gX7Oa05IwVVzHgNjYLD9gMG7SFDQLPEWfsEmgXM31XIBYZq5KJjDwzOOAvExSYiHHyGGBwoHdjBYdgs56LFyJuiWuEeltiwqRceR-zKUa9Xzfk-I36STzIYnJ4Is1Qbv6DFDY_uYQrftW5UJiXKC9Crx7oG6kp2VKVVVM_13rV1cK9NoCK_5U5j6QIw_8uEG74dU6Xsx_aQ3cR_Y8i3SeGI1yb5Z6LarmeU1oNX7yT04h2dm5Lg2IhVGbc1loL6S1rXhRPwIEgyZSply0IyzKPf4rVeccooedNganP07nwEjDQN-imXxsA_nXmwR1Bj_81guiDLvEHAQN6pJBx5McCDxvjJzWUrngSNQj8SARKl5mk7YpNIpM8U__fXACxI-2VqwQKzQ89Sz1F3Wo-3Fijke972cuwdfw76gnJwKaadCwOYI
```

After a bit of research into JWTs, it looks like the `kid` key uses a local network on port `7070`