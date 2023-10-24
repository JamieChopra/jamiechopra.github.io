---
layout: post
title: PG | Fail
subtitle: You shall not pass.
---

# Nmap Scan Results

~~~
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-24 16:26 UTC
Nmap scan report for 192.168.194.126
Host is up (0.015s latency).
Not shown: 65533 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
|_  256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
873/tcp open  rsync   (protocol version 31)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.36 seconds
~~~

# Service Enumeration

## Rsync (TCP/873)
Rsync is a directory sharing protocol used on Unix systems

The protocol can be scanned for available folders using an nmap script
nmap -sV --script "rsync-list-modules" -p 873 192.168.180.126

~~~
PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
| rsync-list-modules: 
|_  fox                 fox home
~~~

# Exploitation

# Privilege Escalation