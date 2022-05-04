![image](https://user-images.githubusercontent.com/64952843/165166129-a70c82e7-12d8-4280-965e-f8cdd97b6913.png)

# [LATE- BOX]  
Hi folks, today I am going to solve an easy rated hack the box machine,Paper created by secnigma.So without any further intro, let's jump in.

# common enumeration

## Nmap
  *TCP over SSH
  *HTTP Default page
  *Host OpenSSH 7.6p1 Ubuntu 4ubuntu0.6
  
#### code-Nmap

```bash
nmap -sC -sV  -A -oN nmap/late 
```


#### output

![[Late/Late png/nmap.png]]

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-26 01:17 EDT
Nmap scan report for 10.129.167.236
Host is up (0.64s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 02:5e:29:0e:a3:af:4e:72:9d:a4:fe:0d:cb:5d:83:07 (RSA)
|   256 41:e1:fe:03:a5:c7:97:c4:d5:16:77:f3:41:0c:e9:fb (ECDSA)
|_  256 28:39:46:98:17:1e:46:1a:1e:a1:ab:3b:9a:57:70:48 (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Late - Best online image tools
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.77 seconds
```

Three ports are open:
port[22]-ssh
port[80]-http

## Default Page-LATE
so lets check the default page 

![[Late/Late png/web.png]]



