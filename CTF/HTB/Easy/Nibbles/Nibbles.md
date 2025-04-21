#easy 

## 0. **Lab infos**

**Difficulty :** Easy
**Name :** Nibbles
**IP** : 10.129.96.84
**Objectives :** Get user and root flags
**OS :** Linux

##### **Conclusion**
**Time :** 
	**Start :** 21.04.2025 / 18:13
	**Break at/to :** 
	**Finish :** 
**Satisfaction :**  
### 1. **Reconnaissance

We start with an #nmap scan :

```BASH
nmap -A 10.129.96.84

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
```

![[Pasted image 20250421181609.png]]

On the inspector we have a comment who says /nibbleblog/

![[Pasted image 20250421181955.png]]

We make a gobuster scan :

```BASH
─[eu-dedivip-1]─[10.10.14.117]─[bobinette@htb-3oaajlzaes]─[~]
└──╼ [★]$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 10.129.96.84/nibbleblog/ -x html,php,db,txt,js,log,config,py,bdd
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.96.84/nibbleblog/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,db,txt,html,log,config,py,bdd,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 303]
/.php                 (Status: 403) [Size: 302]
/index.php            (Status: 200) [Size: 2987]
/sitemap.php          (Status: 200) [Size: 402]
/content              (Status: 301) [Size: 325] [--> http://10.129.96.84/nibbleblog/content/]
/themes               (Status: 301) [Size: 324] [--> http://10.129.96.84/nibbleblog/themes/]
/feed.php             (Status: 200) [Size: 302]
/admin                (Status: 301) [Size: 323] [--> http://10.129.96.84/nibbleblog/admin/]
/admin.php            (Status: 200) [Size: 1401]
/plugins              (Status: 301) [Size: 325] [--> http://10.129.96.84/nibbleblog/plugins/]
/install.php          (Status: 200) [Size: 78]
/update.php           (Status: 200) [Size: 1622]
/README               (Status: 200) [Size: 4628]
/languages            (Status: 301) [Size: 327] [--> http://10.129.96.84/nibbleblog/languages/]
/LICENSE.txt          (Status: 200) [Size: 35148]
```

![[Pasted image 20250421183007.png]]



## 2. **Weaponization

## 3. **Delivery

## 4. **Exploitation

## 5. **Installation

## 6. **Command and Control

## 7. **Actions on Objectives

