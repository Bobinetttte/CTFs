#medium 

## 0. **Lab infos**

**Difficulty :** Medium
**Name :** Whats Your Name?
**IP** : 10.10.219.254
**Objectives :** Get the accessing the moderator account flag and the admin panel flag
**OS :** ?

##### **Conclusion**
**Time :** 
	**Start :** 25.03.2025 18:55
	**Break at/to :** 25.03.2025 18:55 / 
	**Finish :** 
**Satisfaction :**  
### 1. **Reconnaissance

First #nmap scan

```BASH
map -A -p- -v 10.10.219.254
```


Output :

```BASH
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
8081/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
```

It's a web app.

We will analyse the web app with #burpsuite 

On the port 80 we have this sitemap and we have the possibilitie to pre-register.
![[Pasted image 20250325190405.png]]

We can try to register.
One we register we have a login page but when we want to login it's say "User not verified" but we also have this :
![[Pasted image 20250325191031.png]]

But the subdomain doesn't work.

When we enter admin instead of Bobinette it's said "Invalid email or password".

We find some pages with #gobuster 

Output :

```BASH
/.php                 (Status: 403) [Size: 277]
/.html                (Status: 403) [Size: 277]
/index.php            (Status: 200) [Size: 1797]
/login.php            (Status: 200) [Size: 1785]
/register.php         (Status: 200) [Size: 2188]
/admin.php            (Status: 403) [Size: 0]
/upload.php           (Status: 403) [Size: 0]
/logout.php           (Status: 200) [Size: 154]
/mod.php              (Status: 403) [Size: 0]
/dashboard.php        (Status: 403) [Size: 0]
```



On the port 8081 we have nothin execpt a message who let know us that there is a login.php page.

![[Pasted image 20250325190550.png]]

We find some pages with #gobuster 

Output :

```BASH
/.php                 (Status: 403) [Size: 277]
/.html                (Status: 403) [Size: 277]
/index.php            (Status: 200) [Size: 1797]
/login.php            (Status: 200) [Size: 1785]
/register.php         (Status: 200) [Size: 2188]
/admin.php            (Status: 403) [Size: 0]
/upload.php           (Status: 403) [Size: 0]
/logout.php           (Status: 200) [Size: 154]
/mod.php              (Status: 403) [Size: 0]
/dashboard.php        (Status: 403) [Size: 0]
```

## 2. **Weaponization

## 3. **Delivery

## 4. **Exploitation

## 5. **Installation

## 6. **Command and Control

## 7. **Actions on Objectives