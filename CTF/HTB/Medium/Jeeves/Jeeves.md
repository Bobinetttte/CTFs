#medium 

## 0. **Lab infos**

**Difficulty :** Medium
**Name :** Jeeves
**IP** : 10.129.228.112
**Objectives :** Get user and root flags
**OS :** #Windows

##### **Conclusion**
**Time :** 
	**Start :** 22.04.2025 16:31
	**Break at/to :** 
	**Finish :** 
**Satisfaction :**  
### 1. **Reconnaissance

We start with an #nmap scan

```BASH
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Ask Jeeves
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-title: Error 404 Not Found
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
```

And we make a #gobuster scan on the port 50000 because it's the only we can see without the -p-

```BASH
┌─[eu-dedivip-1]─[10.10.14.117]─[bobinette@htb-0k04etkbc3]─[~]
└──╼ [★]$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.129.228.112:50000
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.228.112:50000
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/askjeeves            (Status: 302) [Size: 0] [--> http://10.129.228.112:50000/askjeeves/]
Progress: 108476 / 220561 (49.18%)
```

![[Pasted image 20250422164215.png]]



## 2. **Weaponization

## 3. **Delivery

## 4. **Exploitation

## 5. **Installation

## 6. **Command and Control

## 7. **Actions on Objectives

