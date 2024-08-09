
![logo](/logo.png)

# [Beep- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 15 Mar 2017 as the first machine on HTB,Beep created by ch4p.So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *ssh
  *Elastix
  *Apache 2.2.3
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/beep 10.10.10.7
```

###### Output 
![](/Linux/Linux-Easy/Beep/Screenshots/namapbeep.png)

```sh
 nmap -sV -sC -oA nmap/beep 10.10.10.7                                      ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-09 19:39 EAT
Nmap scan report for 10.10.10.7
Host is up (0.32s latency).
Not shown: 988 closed tcp ports (reset)
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN
80/tcp    open  http       Apache httpd 2.2.3
|_http-title: Did not follow redirect to https://10.10.10.7/
|_http-server-header: Apache/2.2.3 (CentOS)
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: APOP TOP PIPELINING LOGIN-DELAY(0) UIDL EXPIRE(NEVER) RESP-CODES IMPLEMENTATION(Cyrus POP3 server v2) STLS AUTH-RESP-CODE USER
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            790/udp   status
|_  100024  1            793/tcp   status
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: ID IMAP4rev1 SORT=MODSEQ IMAP4 URLAUTHA0001 Completed ACL THREAD=REFERENCES OK X-NETSCAPE CATENATE NO MULTIAPPEND ATOMIC UNSELECT RIGHTS=kxte ANNOTATEMORE LISTEXT NAMESPACE IDLE CONDSTORE STARTTLS LIST-SUBSCRIBED SORT THREAD=ORDEREDSUBJECT LITERAL+ BINARY RENAME MAILBOX-REFERRALS UIDPLUS QUOTA CHILDREN
443/tcp   open  ssl/http   Apache httpd 2.2.3 ((CentOS))
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2017-04-07T08:22:08
|_Not valid after:  2018-04-07T08:22:08
|_ssl-date: 2024-08-09T16:43:52+00:00; +3s from scanner time.
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Elastix - Login page
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql      MySQL (unauthorized)
4445/tcp  open  upnotifyp?
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com

Host script results:
|_clock-skew: 2s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 484.62 seconds

```

looking at the results  we find out that there are 12 ports open and its running an `Apache/2.2.3 (CentOS)`. with a hostname of `beep.localdomain`

port[22]-ssh 
port[25]- smtp
port[80]-http
port[110]-pop3
port[111]-rpcbind
port[143]-imap
port[443]-ssl/http
port[993]-ssl/imap
port[995]-pop3
port[3306]-mysql
port[4445]-upnotifyp
port[1000]-http

now looking for the website in the browser we find that it can not open because of some TLs issue where  Firefox and other browsers changing their min TLS version to a number higher than what the box supports. 

![](/Linux/Linux-Easy/Beep/Screenshots/beeppage.png)

The workaround to this is to edit the `security.tls.version.min` to `1`.
To modify the minimum TLS version in Firefox, follow these steps:
1. Open a new tab in Firefox.Enter `about:config` in the address bar and hit Enter/Return.
2. In the search box located above the list, enter `security.tls.version.min`.
3. Locate the preference with the name `security.tls.version.min` and modify its value to `1`.

![](/Linux/Linux-Easy/Beep/Screenshots/aboutconfig.png)

now when you load the page again you get it working 

![](/Linux/Linux-Easy/Beep/Screenshots/elastic.png)

now after doing so information gathering i now go and intiate a gobuster so that i can be able to enumerate active directories from the website. We run it with `k` flag so as to skip the `tls` verification.

```sh
gobuster dir -u https://10.10.10.7 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -k
```

![](/Linux/Linux-Easy/Beep/Screenshots/gobuster.png)

so we get get some intresting directories we check `admin` but it requires a password and username. images but nothing is of use . `modules` all are empty . `mail` we find that its running `roundcube` in help we find an intresting help screen which clarifies for us more about the system

![](/Linux/Linux-Easy/Beep/Screenshots/help.png)

so when i go and do default password and username it redirects me to `FreePBX` 

![](/Linux/Linux-Easy/Beep/Screenshots/freepbx.png)

so i decide to search from an exploit using the searchsploit to see if i can get some exploit

```sh
searchsplot elastix
```

![](/Linux/Linux-Easy/Beep/Screenshots/elastix.png)

so i see a file inclustion and try to see the exploit so to undrstand what it does. we see that their is an LFI in the `vtigercrm`

![](/Linux/Linux-Easy/Beep/Screenshots/vitigercrm.png)

so when copying the LFI and going to the site we get some html rnder but not in a good format

![](/Linux/Linux-Easy/Beep/Screenshots/lfi.png)

so we decide to view the source so that we can get atleast the good format

![](/Linux/Linux-Easy/Beep/Screenshots/source.png)

so just see the config is showing so credential and let creat a new file and throw every thing that could be a pontential password.
![](/Linux/Linux-Easy/Beep/Screenshots/configpass.png)

it is evident that the machine is vulnerable to password reuse, now let LFI to passwd to get pontential users of the box 

![](/Linux/Linux-Easy/Beep/Screenshots/pottential%20users.png)

now i want to do a bruteforce on the users and the passwords with hydra 

```sh
hydra -L potentialusers.txt -P pass.txt ssh://10.10.10.7 -t 4
```

![](/Linux/Linux-Easy/Beep/Screenshots/bruteforce.png)


Now then what I do i try passwords over the ssh and i get that the password for the root user 


![](/Linux/Linux-Easy/Beep/Screenshots/ssh.png)

from that i navigate to get the user flag 

![](/Linux/Linux-Easy/Beep/Screenshots/userflag.png)
from that i navigate to get the root flag 

![](/Linux/Linux-Easy/Beep/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------