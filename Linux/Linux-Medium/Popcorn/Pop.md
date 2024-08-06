![logo](/logo.png)

# [Popcorn- BOX]  
Hi folks, today I am going to solve an medium rated hack the box machine which was released on 15 Mar 2017 as the fourth machine on HTB,Popcorn created by ch4p.So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *Ftp and ssh
  *Samba
  *Unix Debian
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/popcorn 10.10.10.6
```


###### Output 

![](/Linux/Linux-Medium/Popcorn/Screenshots/nmappopcorn.png)


```sh
nmap -sV -sC -oA nmap/popcorn 10.10.10.6                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-04 18:37 EAT
Nmap scan report for 10.10.10.6
Host is up (0.36s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.1p1 Debian 6ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 3e:c8:1b:15:21:15:50:ec:6e:63:bc:c5:6b:80:7b:38 (DSA)
|_  2048 aa:1f:79:21:b8:42:f4:8a:38:bd:b8:05:ef:1a:07:4d (RSA)
80/tcp open  http    Apache httpd 2.2.12
|_http-server-header: Apache/2.2.12 (Ubuntu)
|_http-title: Did not follow redirect to http://popcorn.htb/
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.33 seconds

```

looking at the results  we find out that there are 2 ports open and its a `Debian 6ubuntu2 `and its running an `Apache server`. with a hostname of `popcorn.htb`

port[22]-ssh 
port[80]- Apache httpd 2.2.12

we need to add the hostname to ``/etc/hosts ``file and browse the page.

#### code-/etc/hosts

```bash
echo 10.10.10.6 popcorn.htb > /etc/hosts
```

so let open the page [popcorn.htb](http://popcorn.htb/) loading the website reveals only the default `Apache` page.

![](/Linux/Linux-Medium/Popcorn/Screenshots/popcornbrowser.png)

Now since this is what we can view there is no much that we can do on the browser so let us find other directories from the server by enumerating with `gobuster`

###### code

```sh
gobuster dir -u http://popcorn.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
```

###### output

![](/Linux/Linux-Medium/Popcorn/Screenshots/gobusterpopcorn.png)

![](/Linux/Linux-Medium/Popcorn/Screenshots/torrentpopcorn.png)

looking at the status code i try all with code `200` and `301` and find something intresting  on the [torrent](http://popcorn.htb/torrent/)
