#medium

## 0. **Lab infos**

**Difficulty :** medium
**Name :** Lightweight
**IP** : 10.129.9.192
**Objectives :** Get the Idapuser2 flag in the home directory and the root flag in the home directory
**OS :** Linux

##### **Conclusion**
**Time :** 
	**Start :** 24.04.2025 18:26
	**Break at/to :** 
	**Finish :**
**Satisfaction :**  
### 1. **Reconnaissance**

We start with an #nmap scan :

```BASH
┌─[eu-dedivip-1]─[10.10.14.145]─[bobinette@htb-yrfyly7wla]─[~]
└──╼ [★]$ nmap -A 10.129.9.192
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-24 11:27 CDT
Nmap scan report for 10.129.9.192
Host is up (0.0070s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 19:97:59:9a:15:fd:d2:ac:bd:84:73:c4:29:e9:2b:73 (RSA)
|   256 88:58:a1:cf:38:cd:2e:15:1d:2c:7f:72:06:a3:57:67 (ECDSA)
|_  256 31:6c:c1:eb:3b:28:0f:ad:d5:79:72:8f:f5:b5:49:db (ED25519)
80/tcp  open  http    Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips mod_fcgid/2.3.9 PHP/5.4.16)
|_http-title: Lightweight slider evaluation page - slendr
389/tcp open  ldap    OpenLDAP 2.2.X - 2.3.X
| ssl-cert: Subject: commonName=lightweight.htb
| Subject Alternative Name: DNS:lightweight.htb, DNS:localhost, DNS:localhost.localdomain
| Not valid before: 2018-06-09T13:32:51
|_Not valid after:  2019-06-09T13:32:51
|_ssl-date: TLS randomness does not represent time
```

![[Pasted image 20250424183007.png]]

Now gobuster :


## 2. **Weaponization**

## 3. **Delivery**

## 4. **Exploitation**

## 5. **Installation**

## 6. **Command and Control**

## 7. **Actions on Objectives**

