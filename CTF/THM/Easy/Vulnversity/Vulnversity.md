
## 0. **Lab infos**

**Difficulty :** #easy 
**Name :** Vulnversity
**IP** : 10.10.216.247
**Objectives :** Answer all the questions and get the user and root flag
**OS :** ?

##### **Conclusion**
**Time :** 
	**Start :** 04/05/2025 16:35
	**Break at/to :** 
	**Finish :**
**Satisfaction :** 
## 1. **Reconnaissance**

We start with an #nmap scan :

```BASH
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
|   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
|_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Vuln University
```


And now a gobuster scan :

```BASH
root@ip-10-10-16-93:~# gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.216.247:3333 -x html,php,txt,py,log,conf,db
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.216.247:3333
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,txt,py,log,conf,db
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 295]
/index.html           (Status: 200) [Size: 33014]
/.php                 (Status: 403) [Size: 294]
/images               (Status: 301) [Size: 322] [--> http://10.10.216.247:3333/images/]
/css                  (Status: 301) [Size: 319] [--> http://10.10.216.247:3333/css/]
/js                   (Status: 301) [Size: 318] [--> http://10.10.216.247:3333/js/]
/fonts                (Status: 301) [Size: 321] [--> http://10.10.216.247:3333/fonts/]
/internal             (Status: 301) [Size: 324] [--> http://10.10.216.247:3333/internal/]
Progress: 731339 / 1746208 (41.88%)
```

![[Pasted image 20250504165126.png]]


## 2. **Weaponization**

## 3. **Delivery**

## 4. **Exploitation**

Now we are going to bruteforce the extensions of the file

## 5. **Installation**

## 6. **Command and Control**

## 7. **Actions on Objectives**

