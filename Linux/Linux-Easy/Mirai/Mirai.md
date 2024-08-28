![logo](/logo.png)

# [Mirai- BOX]  
Hi folks, today I am going to solve an Easy rated hack the box machine which was released on 01 Sep 2017 as the tenth machine on HTB,Mirai created by [Arrexel](https://app.hackthebox.com/users/2904) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *plex
  *pi-hole
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/mirai 10.10.10.48
```

###### Output 

![](/Linux/Linux-Easy/Mirai/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/mirai 10.10.10.48                                                                                           ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-28 18:34 EAT
Nmap scan report for 10.10.10.48
Host is up (0.37s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey: 
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (ED25519)
53/tcp open  domain  dnsmasq 2.76
| dns-nsid: 
|_  bind.version: dnsmasq-2.76
80/tcp open  http    lighttpd 1.4.35
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: lighttpd/1.4.35
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.71 seconds
```

looking at the results  we find out that there are 3 ports open and its a `Debian`and its running an `lighttpd`. 


port[22]-ssh
port[53]-dns
port[80]-  lightttpd 1.4.35

So when we go the browser at [http://10.10.10.48](http://10.10.10.48)  looking at the browser we find an empty page

![](/Linux/Linux-Easy/Mirai/Screenshots/browser.png)

I start `gobuster` and run it  to enumerate existing directories.

```sh
gobuster dir -u http://10.10.10.48 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -k  
```

![](/Linux/Linux-Easy/Mirai/Screenshots/gobuster.png)

we find something intresting an admin page which seem to be running a Pi-hole admin dashboard From here, it is safe to assume that the target is a Raspberry Pi machine, and is most likely running Raspbian.

![](/Linux/Linux-Easy/Mirai/Screenshots/Pi-hole.png)

# Exploitation

Knowing the target operating system and device, while keeping in mind how the Mirai botnet.
[Mirai](https://en.wikipedia.org/wiki/Mirai_(malware)) is a real malware that formed a huge network of bots, and is used to conduct distributed denial of service (DDOS) attacks. The compromised devices are largely made up of internet of things (IoT) devices running embedded processors like ARM and MIPS. The most famous Mirai attack was in October 2016, when the botnet degraded the service of Dyn, a DNS service provider, which resulted in making major sites across the internet (including NetFlix, Twitter, and GitHub) inaccessible. The sites were still up, but without DNS, no one could access them.

Mirai’s go-to attack was to brute force common default passwords. In fact, `mirai-botnet.txt` was added to [SecLists](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Malware/mirai-botnet.txt) in November 2017.

it can be assumed that the `default user credentials` have been unchanged. A quick
search reveals that the default `Raspbian credentials` are `pi:raspberry`. Connecting via SSH with these credentials immediately gives full access to the device, as the default configuration for
Raspbian has the `pi `user as part of the `sudoers group`.

![](/Linux/Linux-Easy/Mirai/Screenshots/Shell.png)

I use `tree .`  to get to know where the user flag is 

![](/Linux/Linux-Easy/Mirai/Screenshots/tree.png)

you can then obtain your `user flag`

![](/Linux/Linux-Easy/Mirai/Screenshots/pi.png)

Upon closer inspection, the root flag is not in its typical location. Instead, the root.txt files presents the message `I lost my original root.txt! I think I may have a backup on my USB stick…` 

![](/Linux/Linux-Easy/Mirai/Screenshots/root.png)

# Privillage Escalation

While this machine does not require any exploitation to obtain root permissions, the flag must be
obtained through alternate methods. Based on the hint in the `root.txt` file, it can be assumed that there is a mounted drive or partition that contains a copy of the original file. Running `df -h` outputs a list of the machine’s partitions, the last of which being mounted on `/media/usbstick`.

![](/Linux/Linux-Easy/Mirai/Screenshots/dfh.png)

Browsing to /media/usbstick, there is a single file, `damnit.txt`. The contents are:

![](/Linux/Linux-Easy/Mirai/Screenshots/damit.png)

The raw USB device is `/dev/sdb`, and I can interact with that just like any other file. I’ll show a couple different ways to recover the flag. By running `strings` on the file `sdb` it revels the root.

![](Linux/Linux-Easy/Mirai/Screenshots/strings.png)

#### Method 2

Lets copy the image to ur local host and try recover it but first lets copy the image to the home directory

```sh
sudo dcfldd if=/dev/sdb of=/home/pi/usb.dd
```

![](/Linux/Linux-Easy/Mirai/Screenshots/image.png)

then lets use the `Scp` to copy the file to our local host

```sh
scp pi@10.10.10.48:/home/pi/usb.dd .
```

![](/Linux/Linux-Easy/Mirai/Screenshots/copied.png)

`extundelete` is a [data recovery utility](http://extundelete.sourceforge.net/) that works here to recover `root.txt`. I’ll install it (`sudo apt install extundelete`) and then run it with the `--restore-all` flag

![](/Linux/Linux-Easy/Mirai/Screenshots/extundelete.png)

and you get your `root flag`

![](/Linux/Linux-Easy/Mirai/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------




