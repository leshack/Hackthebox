![logo](/logo.png)

# [Popcorn- BOX]  
Hi folks, today I am going to solve an medium rated hack the box machine which was released on 15 Mar 2017 as the fourth machine on HTB,Popcorn created by ch4p.So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *Ftp and ssh
  *Samba
  *Unix Debian
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/popcorn 10.10.10.6
```


###### Output 

![](/Linux/Linux-Medium/Popcorn/Screenshots/nmappopcorn.png)


```sh
nmap -sV -sC -oA nmap/popcorn 10.10.10.6                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-04 18:37 EAT
Nmap scan report for 10.10.10.6
Host is up (0.36s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.1p1 Debian 6ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 3e:c8:1b:15:21:15:50:ec:6e:63:bc:c5:6b:80:7b:38 (DSA)
|_  2048 aa:1f:79:21:b8:42:f4:8a:38:bd:b8:05:ef:1a:07:4d (RSA)
80/tcp open  http    Apache httpd 2.2.12
|_http-server-header: Apache/2.2.12 (Ubuntu)
|_http-title: Did not follow redirect to http://popcorn.htb/
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.33 seconds

```

looking at the results  we find out that there are 2 ports open and its a `Debian 6ubuntu2 `and its running an `Apache server`. with a hostname of `popcorn.htb`

port[22]-ssh 
port[80]- Apache httpd 2.2.12

we need to add the hostname to ``/etc/hosts ``file and browse the page.

#### code-/etc/hosts

```bash
echo 10.10.10.6 popcorn.htb > /etc/hosts
```

so let open the page [popcorn.htb](http://popcorn.htb/) loading the website reveals only the default `Apache` page.

![](/Linux/Linux-Medium/Popcorn/Screenshots/popcornbrowser.png)

Now since this is what we can view there is no much that we can do on the browser so let us find other directories from the server by enumerating with `gobuster`

###### code

```sh
gobuster dir -u http://popcorn.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
```

###### output

![](/Linux/Linux-Medium/Popcorn/Screenshots/gobusterpopcorn.png)

![](/Linux/Linux-Medium/Popcorn/Screenshots/torrentpopcorn.png)

looking at the status code i try all with code `200` and `301` and find something intresting  on the [torrent](http://popcorn.htb/torrent/)


![](/Linux/Linux-Medium/Popcorn/Screenshots/torrentwebpopcorn.png)

# Exploitation
Looking around the torrent site a bit, we find that there are module which seem intresting and we can register so that we can explore further.But lets first search for any exploit of `torrent hoster` with `searchsploit`.

###### code

```sh
searchsploit torrent hoster 
```

###### Output

![](/Linux/Linux-Medium/Popcorn/Screenshots/searchsploitpopcorn.png)

We then examine the exploit to see what it does.

![](/Linux/Linux-Medium/Popcorn/Screenshots/examinepopcorn.png)

```sh
========================================================================================
| # Title    : Torrent Hoster Remont Upload Exploit
| # Author   : El-Kahina
| # Home     : www.h4kz.com                                                                              |
| # Script   : Powered by Torrent Hoster.
| # Tested on: windows SP2 Fran&#65533;ais V.(Pnx2 2.0) + Lunix Fran&#65533;ais v.(9.4 Ubuntu)
| # Bug      : Upload
|
====================== Exploit By El-Kahina =================================
 # Exploit  :

 1 - use tamper data :

 http://127.0.0.1/torrenthoster//torrents.php?mode=upload

 2-
    <center>
   Powered by Torrent Hoster
        <br />
        <form enctype="multipart/form-data" action="http://127.0.0.1/torrenthoster/upload.php" id="form" method="post" onsubmit="a=document.getElementById('form').style;a.display='none';b=document.getElementById('part2').style;b.display='inline';" style="display: inline;">
        <strong>&#65533;&#65533;&#65533;&#65533; &#65533;&#65533;&#65533; &#65533;&#65533;&#65533;&#65533;&#65533; &#65533;&#65533; &#65533;&#65533;:</strong> <?php echo $maxfilesize; ?>&#65533;&#65533;&#65533;&#65533;&#65533;&#65533;&#65533;&#65533;<br />
<br>
        <input type="file" name="upfile" size="50" /><br />
<input type="submit" value="&#65533;&#65533;&#65533; &#65533;&#65533;&#65533;&#65533;&#65533;" id="upload" />

```

The most promising option is the Upload section as from the exploit and best from the website it requires authorization.Luckily, there are no restrictions on account creation.

![](/Linux/Linux-Medium/Popcorn/Screenshots/hosterpopcorn.png)

After creating the account we log on to the upload and we find the instruction of uploading a torrent file we then we try uploading an image but it fails so we then download kali torrent to upload it so that we can see how it does it from `burpsuite`

![](/Linux/Linux-Medium/Popcorn/Screenshots/kalitorrentpopcorn.png)

After uploading the kali torrent we get a new page that it intresting where there is a `screenshot` 

![](/Linux/Linux-Medium/Popcorn/Screenshots/torrentkalipopcorn.png)

When we edit the torrent we find that there is a place where we can upload a screenshot image 
![](/Linux/Linux-Medium/Popcorn/Screenshots/screenshottorrent.png)

When we upload an image we get a success message of the image 

![](/Linux/Linux-Medium/Popcorn/Screenshots/successimagepopcorn.png)

so when we navigate to [http://popcorn.htb/torrent/upload/](http://popcorn.htb/torrent/upload/) we find that our image has been uploaded 

![](/Linux/Linux-Medium/Popcorn/Screenshots/imageuploadedpopcorn.png)

so lets go try a php file  

```sh
echo "<?php echo "Hello, World!"; ?>" > popcorn.php
```

After Uploading we get an Invalid File item 

![](/Linux/Linux-Medium/Popcorn/Screenshots/invalidfile.png)

So we open burpsuite to see how its doing the upload of the image  and we find that the content type is `image/png` 

![](/Linux/Linux-Medium/Popcorn/Screenshots/burpimagepopcorn.png)

so we can see the request of the php and be able to modify it so that we can be able to pass the filters but we would need a bit of the image `bytes` this is how the php request is from `burpsuite`
