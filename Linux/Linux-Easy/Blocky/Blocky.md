![logo](/logo.png)

# [Blocky- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 21 Jul 2017 as the tenth machine on HTB,Blocky created by [Arrexel](https://app.hackthebox.com/users/2904).So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *PhpMyAdmin
  *Wordpress
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/blocky 10.10.10.37
```

###### Output 

![](/Linux/Linux-Easy/Blocky/Screenshots/nmap.png)

```sh
 nmap -sV -sC -oA nmap/blocky 10.10.10.37                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-27 18:50 EAT
Nmap scan report for 10.10.10.37
Host is up (0.33s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT     STATE  SERVICE VERSION
21/tcp   open   ftp     ProFTPD 1.3.5a
22/tcp   open   ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
80/tcp   open   http    Apache httpd 2.4.18
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Did not follow redirect to http://blocky.htb
8192/tcp closed sophos
Service Info: Host: 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.85 seconds
```


looking at the results  we find out that there are 4 ports open and its a `Ubuntu`and its running an `Apache server`. 

port[21]-ftp
port[22]-ssh
port[80]-  httpd 2.4.18
port[8192]-sophos

so when we go the browser at [http://10.10.10.37](http://10.10.10.37)  we first add the hostname `blocky.htb` to `/etc/hosts` .

![](/Linux/Linux-Easy/Blocky/Screenshots/browser.png)

it is wordpress website so lets run `wpscan` in the background

```sh
wpscan --url http://blocky.htb -e ap --plugins-detection aggressive  --disable-tls-checks --ignore-main-redirect
```

![](/Linux/Linux-Easy/Blocky/Screenshots/wordpress.png)

I also start `gobuster` and run it on the background to enumerste existing directories.

```sh
gobuster dir -u http://blocky.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -k  
```

![](/Linux/Linux-Easy/Blocky/Screenshots/gobuster.png)

in `plugins` directory we find something interesting two files

![](/Linux/Linux-Easy/Blocky/Screenshots/plugins.png)

# Exploitation

Looking at the jar files, griefprevention is an open source plugin that is freely available.
`BlockyCore`, however, appears to be created by the server administrator, as its title relates
directly to the server.

![](/Linux/Linux-Easy/Blocky/Screenshots/strings.png)

we find credentials of the phpmyadmin lets log in to see the wordpress database

![](/Linux/Linux-Easy/Blocky/Screenshots/phpmyadmin.png)

we find some user credential from the users list 

![](/Linux/Linux-Easy/Blocky/Screenshots/userlist.png)

The intended method for this machine is a simple `username` and `password reuse`. Attempting to
connect via SSH to the `notch` user

![](/Linux/Linux-Easy/Blocky/Screenshots/shell.png)

we can find the `user` flag at 

![](/Linux/Linux-Easy/Blocky/Screenshots/userflag.png)

as you can see above `notch` is from the sudo group 

```sh
sudo i
```

![](/Linux/Linux-Easy/Blocky/Screenshots/sudouser.png)

and just like that we get the root flag

	-------------------------END successful attack @lesley----------------------

