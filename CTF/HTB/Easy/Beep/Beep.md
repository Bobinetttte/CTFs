#easy

## 0. **Lab infos**

**Difficulty :** Easy
**Name :** Beep
**IP** : 10.129.229.183
**Objectives :** Get user and root flags
**OS :** Linux

##### **Conclusion**
**Time :** 
	**Start :** 04.04.2025 / 18:39
	**Break at/to :** 04.04.2025 / 19:17 | 06.04.2025 / 19:30
	**Finish :** 
**Satisfaction :**  
### 1. **Reconnaissance

We start with an #nmap scan :

```BASH
nmap -A -v -p- 10.129.229.183

PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp?
|_smtp-commands: Couldn't establish connection on port 25
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://10.129.229.183/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
110/tcp   open  pop3?
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            853/udp   status
|_  100024  1            856/tcp   status
143/tcp   open  imap?
443/tcp   open  ssl/https?
|_ssl-date: 2025-04-04T16:45:04+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Issuer: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2017-04-07T08:22:08
| Not valid after:  2018-04-07T08:22:08
| MD5:   621a:82b6:cf7e:1afa:5284:1c91:60c8:fbc8
|_SHA-1: 800a:c6e7:065e:1198:0187:c452:0d9b:18ef:e557:a09f
856/tcp   open  status     1 (RPC #100024)
993/tcp   open  imaps?
995/tcp   open  pop3s?
3306/tcp  open  mysql?
4190/tcp  open  sieve?
4445/tcp  open  upnotifyp?
4559/tcp  open  hylafax?
5038/tcp  open  asterisk   Asterisk Call Manager 1.1
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-server-header: MiniServ/1.570
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: F3337C71F21F2D6F478E118940F48988
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
```


We make a scan with #gobuster 

```BASH
gobuster dir -u https://10.129.229.183 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,htm,js,txt,log,config,logs,conf,py,db -k
```
-k is for authorizing the expired SSL certificate

```BASH
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htm                 (Status: 403) [Size: 286]
/.html                (Status: 403) [Size: 287]
/index.php            (Status: 200) [Size: 1785]
/images               (Status: 301) [Size: 318] [--> https://10.129.229.183/images/]
/help                 (Status: 301) [Size: 316] [--> https://10.129.229.183/help/]
/register.php         (Status: 200) [Size: 1785]
/themes               (Status: 301) [Size: 318] [--> https://10.129.229.183/themes/]
/modules              (Status: 301) [Size: 319] [--> https://10.129.229.183/modules/]
/mail                 (Status: 301) [Size: 316] [--> https://10.129.229.183/mail/]
/admin                (Status: 301) [Size: 317] [--> https://10.129.229.183/admin/]
/static               (Status: 301) [Size: 318] [--> https://10.129.229.183/static/]
/lang                 (Status: 301) [Size: 316] [--> https://10.129.229.183/lang/]
/config.php           (Status: 200) [Size: 1785]
/robots.txt           (Status: 200) [Size: 28]
/var                  (Status: 301) [Size: 315] [--> https://10.129.229.183/var/]
```

When I enter /admin I got an alert for login and when I click cancel Its sen me there :
![[Pasted image 20250404190403.png]]

We can see that we are on FreePBX 2.8.1.4
We find the CVE-2010-3490
The CVE is not working.

So we start to find if some vulnerabilities with elastix exist and we found this :
![[Pasted image 20250406202904.png]]


## 2. **Weaponization

## 3. **Delivery

## 4. **Exploitation

So let's try the #LFI exploit we find with Elatix.

![[Pasted image 20250406203046.png]]

It's working and we have the following informations :

-  DB :
    
    - `AMPDBUSER=asteriskuser`
        
    - `AMPDBPASS=jEhdIekWmdjE`
        
-  **FreePBX** :
    
    - `AMPMGRUSER=admin`
        
    - `AMPMGRPASS=jEhdIekWmdjE`
        
    - `ARI_ADMIN_USERNAME=admin`

![[Pasted image 20250406203824.png]]

Now we want to connect with ssh :

`ssh admin@10.129.229.183`

Output : `Unable to negotiate with 10.129.229.183 port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1`


So we use :

`ssh -oKexAlgorithms=+diffie-hellman-group14-sha1 -oHostKeyAlgorithms=+ssh-rsa admin@10.129.229.183`

Output

## 5. **Installation

## 6. **Command and Control

## 7. **Actions on Objectives
