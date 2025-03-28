## 0. **Lab infos**

**Difficulty :** #Hard
**Name :** El Bandito
**IP** : 10.10.66.64
**Objectives :** Get the first web flag and the second web flag.
**OS :** ?

##### **Conclusion**
**Time :** 
	**Start :** 28.03.2025 18:10
	**Break at/to :** 28.03.2025 19:27
	**Finish :** 
**Satisfaction :**  
### 1. **Reconnaissance

We start with an #nmap scan :

```BASH
nmap -A -p- -v elbandito.thm

PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
80/tcp   open  ssl/http El Bandito Server
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 NOT FOUND
|     Date: Fri, 28 Mar 2025 17:12:11 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 207
|     Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none';
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: SAMEORIGIN
|     X-XSS-Protection: 1; mode=block
|     Feature-Policy: microphone 'none'; geolocation 'none';
|     Age: 0
|     Server: El Bandito Server
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>404 Not Found</title>
|     <h1>Not Found</h1>
|     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Fri, 28 Mar 2025 17:11:21 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 58
|     Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none';
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: SAMEORIGIN
|     X-XSS-Protection: 1; mode=block
|     Feature-Policy: microphone 'none'; geolocation 'none';
|     Age: 0
|     Server: El Bandito Server
|     Accept-Ranges: bytes
|     Connection: close
|     nothing to see <script src='/static/messages.js'></script>
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Fri, 28 Mar 2025 17:11:21 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 0
|     Allow: GET, HEAD, OPTIONS, POST
|     Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none';
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: SAMEORIGIN
|     X-XSS-Protection: 1; mode=block
|     Feature-Policy: microphone 'none'; geolocation 'none';
|     Age: 0
|     Server: El Bandito Server
|     Accept-Ranges: bytes
|     Connection: close
|   RTSPRequest: 
|_    HTTP/1.1 400 Bad Request
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS POST
|_http-server-header: El Bandito Server
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
| ssl-cert: Subject: commonName=localhost
| Subject Alternative Name: DNS:localhost
| Issuer: commonName=localhost
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-04-10T06:51:56
| Not valid after:  2031-04-08T06:51:56
| MD5:   b27d 890a f042 b213 5197 0a59 8e7d 7032
|_SHA-1: c7ef 94a2 c156 0d67 0228 3e0e ec9d 208b 1ef7 2610
631/tcp  open  ipp      CUPS 2.4
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: CUPS/2.4 IPP/2.1
|_http-title: Bad Request - CUPS v2.4.7
8080/tcp open  http     nginx
|_http-favicon: Unknown favicon MD5: 0488FACA4C19046B94D07C3EE83CF9D6
|_http-title: Site doesn't have a title (application/json;charset=UTF-8).
```

There is the following ports :

```PORTS
22
80
631
8080
```

Port `8080`
![[Pasted image 20250328182102.png]]
![[Pasted image 20250328182316.png]]
![[Pasted image 20250328182920.png]]

We make a scan we #gobuster :

```BASH
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://elbandito.thm:8080/ -x html,php,js,db,log,txt,py,htm,conf,config
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://elbandito.thm:8080/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              conf,config,html,txt,py,log,htm,php,js,db
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 557]
/index.htm            (Status: 200) [Size: 557]
/services.html        (Status: 200) [Size: 3791]
/info                 (Status: 200) [Size: 2]
/admin                (Status: 403) [Size: 146]
/admin.html           (Status: 403) [Size: 146]
/admin.txt            (Status: 403) [Size: 146]
/admin.py             (Status: 403) [Size: 146]
/admin.config         (Status: 403) [Size: 146]
/admin.conf           (Status: 403) [Size: 146]
/admin.php            (Status: 403) [Size: 146]
/admin.js             (Status: 403) [Size: 146]
/admin.db             (Status: 403) [Size: 146]
/admin.htm            (Status: 403) [Size: 146]
/admin.log            (Status: 403) [Size: 146]
/health               (Status: 200) [Size: 150]
/assets               (Status: 200) [Size: 0]
/traceroute           (Status: 403) [Size: 146]
/traceroute.php       (Status: 403) [Size: 146]
/traceroute.js        (Status: 403) [Size: 146]
/traceroute.db        (Status: 403) [Size: 146]
/traceroute.log       (Status: 403) [Size: 146]
/traceroute.htm       (Status: 403) [Size: 146]
/traceroute.html      (Status: 403) [Size: 146]
/traceroute.txt       (Status: 403) [Size: 146]
/traceroute.py        (Status: 403) [Size: 146]
/traceroute.conf      (Status: 403) [Size: 146]
/traceroute.config    (Status: 403) [Size: 146]
/trace                (Status: 403) [Size: 146]
/trace.js             (Status: 403) [Size: 146]
/trace.db             (Status: 403) [Size: 146]
/trace.log            (Status: 403) [Size: 146]
/trace.htm            (Status: 403) [Size: 146]
/trace.php            (Status: 403) [Size: 146]
/trace.py             (Status: 403) [Size: 146]
/trace.txt            (Status: 403) [Size: 146]
/trace.config         (Status: 403) [Size: 146]
/trace.conf           (Status: 403) [Size: 146]
/trace.html           (Status: 403) [Size: 146]
/environment          (Status: 403) [Size: 146]
/environment.py       (Status: 403) [Size: 146]
/environment.conf     (Status: 403) [Size: 146]
/environment.config   (Status: 403) [Size: 146]
/environment.html     (Status: 403) [Size: 146]
/environment.txt      (Status: 403) [Size: 146]
/environment.db       (Status: 403) [Size: 146]
/environment.log      (Status: 403) [Size: 146]
/environment.htm      (Status: 403) [Size: 146]
/environment.php      (Status: 403) [Size: 146]
/environment.js       (Status: 403) [Size: 146]
/administration       (Status: 403) [Size: 146]
/administration.js    (Status: 403) [Size: 146]
/administration.php   (Status: 403) [Size: 146]
/administration.db    (Status: 403) [Size: 146]
/administration.log   (Status: 403) [Size: 146]
/administration.htm   (Status: 403) [Size: 146]
/administration.html  (Status: 403) [Size: 146]
/administration.py    (Status: 403) [Size: 146]
/administration.txt   (Status: 403) [Size: 146]
/administration.conf  (Status: 403) [Size: 146]
/administration.config (Status: 403) [Size: 146]
/envelope_small       (Status: 403) [Size: 146]
/envelope_small.txt   (Status: 403) [Size: 146]
/envelope_small.py    (Status: 403) [Size: 146]
/envelope_small.conf  (Status: 403) [Size: 146]
/envelope_small.db    (Status: 403) [Size: 146]
/envelope_small.js    (Status: 403) [Size: 146]
/envelope_small.html  (Status: 403) [Size: 146]
/envelope_small.config (Status: 403) [Size: 146]
/envelope_small.log   (Status: 403) [Size: 146]
/envelope_small.php   (Status: 403) [Size: 146]
/envelope_small.htm   (Status: 403) [Size: 146]
```



Port `80`
![[Pasted image 20250328182120.png]]
![[Pasted image 20250328182607.png]]



Port `631`
![[Pasted image 20250328182141.png]]
![[Pasted image 20250328182743.png]]



## 2. **Weaponization

## 3. **Delivery

## 4. **Exploitation

## 5. **Installation

## 6. **Command and Control

## 7. **Actions on Objectives

