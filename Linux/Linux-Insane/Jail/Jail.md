![logo](/logo.png)

# [Jail- BOX]  
Hi folks, today I am going to solve an Insane rated hack the box machine which was released on 14 Jul 2017 as the 22nd machine on HTB,Jail created by [n0decaf](https://app.hackthebox.com/users/250) without any further intro, let'sf jump in.

# common enumeration

## Nmap
  *http
  *ssh
  *OverflowBuffer
###### code-nmap

```code
nmap -sV -sC -oA nmap/jail 10.10.10.34
```

###### Output 

![](/Linux/Linux-Insane/Jail/Screenshots/nmap.png)

```sh
nmap -sV -sC -oA nmap/jail 10.10.10.34                                                                                            ─╯
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-26 18:43 EAT
Nmap scan report for 10.10.10.34
Host is up (0.37s latency).
Not shown: 969 filtered tcp ports (no-response), 27 filtered tcp ports (host-prohibited)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1 (protocol 2.0)
| ssh-hostkey: 
|   2048 cd:ec:19:7c:da:dc:16:e2:a3:9d:42:f3:18:4b:e6:4d (RSA)
|   256 af:94:9f:2f:21:d0:e0:1d:ae:8e:7f:1d:7b:d7:42:ef (ECDSA)
|_  256 6b:f8:dc:27:4f:1c:89:67:a4:67:c5:ed:07:53:af:97 (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS))
|_http-server-header: Apache/2.4.6 (CentOS)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-methods: 
|_  Potentially risky methods: TRACE
111/tcp  open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100003  3,4         2049/udp   nfs
|   100003  3,4         2049/udp6  nfs
|   100005  1,2,3      20048/tcp   mountd
|   100005  1,2,3      20048/tcp6  mountd
|   100005  1,2,3      20048/udp   mountd
|_  100005  1,2,3      20048/udp6  mountd
2049/tcp open  nfs     3-4 (RPC #100003)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 48.00 seconds
```

looking at the results  we find out that there are 4 ports open and its `centos`and its running an `Apache httpd 2.4.6`.  

port[22]-ssh
port[80]-tcp
port[111]-rpcbind 2-4
port[2049]-nfs

when we naviagate to [http://10.10.10.34](http://10.10.10.34)  we find a page of Jail

![](/Linux/Linux-Insane/Jail/Screenshots/brwser.png)

i start `gobuster` to enemerate directories sinces it seems that i can not find anything 

```sh
gobuster dir -u http://10.10.10.34 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -x php -t 50
```


![](/Linux/Linux-Insane/Jail/Screenshots/gobuster.png)

i find a `jailuser` directory

![](/Linux/Linux-Insane/Jail/Screenshots/jailuser.png)

The file contains source code and a binary compiled from the given source. This binary is running as a service on `port 7411`.

### NFSShare

```sh
showmount -e 10.10.10.34
```

![](/Linux/Linux-Insane/Jail/Screenshots/showmount.png)

The `*` are the ACLs as far as what can mount the share. It could have whitelisted IPs/domains, but in this case, anyone can mount them.

i’ve got the three files from `/jailuser`. `compile.sh` compiles the source and installs it in place, restarting the service

```sh
gcc -o jail jail.c -m32 -z execstack
service jail stop
cp jail /usr/local/bin/jail
service jail start
```

It is using `execstack`, which means that data execution prevention (DEP) is disabled, and if I can overflow a buffer, I can write shellcode directly to the stack.

`jail` is a 32-bit ELF executable, presumably the one generated by `jail.c` and `compile.sh`

![](Linux/Linux-Insane/Jail/Screenshots/jailbinary.png)

in `Jail.c`

```sh
#include <stdio.h>
#include <stdlib.h>
#include <netdb.h>
#include <netinet/in.h>
#include <string.h>
#include <unistd.h>
#include <time.h>

int debugmode;
int handle(int sock);
int auth(char *username, char *password);

int auth(char *username, char *password) {
    char userpass[16];
    char *response;
    if (debugmode == 1) {
        printf("Debug: userpass buffer @ %p\n", userpass);
        fflush(stdout);
    }
    if (strcmp(username, "admin") != 0) return 0;
    strcpy(userpass, password);
    if (strcmp(userpass, "1974jailbreak!") == 0) {
        return 1;
    } else {
        printf("Incorrect username and/or password.\n");
        return 0;
    }
    return 0;
}

int handle(int sock) {
    int n;
    int gotuser = 0;
    int gotpass = 0;
    char buffer[1024];
    char strchr[2] = "\n\x00";
    char *token;
    char username[256];
    char password[256];
    debugmode = 0;
    memset(buffer, 0, 256);
    dup2(sock, STDOUT_FILENO);
    dup2(sock, STDERR_FILENO);
    printf("OK Ready. Send USER command.\n");
    fflush(stdout);
    while(1) {
        n = read(sock, buffer, 1024);
        if (n < 0) {
            perror("ERROR reading from socket");
            return 0;
        }
        token = strtok(buffer, strchr);
        while (token != NULL) {
            if (gotuser == 1 && gotpass == 1) {
                break;
            }
            if (strncmp(token, "USER ", 5) == 0) {
                strncpy(username, token+5, sizeof(username));
                gotuser=1;
                if (gotpass == 0) {
                    printf("OK Send PASS command.\n");
                    fflush(stdout);
                }
            } else if (strncmp(token, "PASS ", 5) == 0) {
                strncpy(password, token+5, sizeof(password));
                gotpass=1;
                if (gotuser == 0) {
                    printf("OK Send USER command.\n");
                    fflush(stdout);
                }
            } else if (strncmp(token, "DEBUG", 5) == 0) {
                if (debugmode == 0) {
                    debugmode = 1;
                    printf("OK DEBUG mode on.\n");
                    fflush(stdout);
                } else if (debugmode == 1) {
                    debugmode = 0;
                    printf("OK DEBUG mode off.\n");
                    fflush(stdout);
                }
            }
            token = strtok(NULL, strchr);
        }
        if (gotuser == 1 && gotpass == 1) {
            break;
        }
    }
    if (auth(username, password)) {
        printf("OK Authentication success. Send command.\n");
        fflush(stdout);
        n = read(sock, buffer, 1024);
        if (n < 0) {
            perror("Socket read error");
            return 0;
        }
        if (strncmp(buffer, "OPEN", 4) == 0) {
            printf("OK Jail doors opened.");
            fflush(stdout);
        } else if (strncmp(buffer, "CLOSE", 5) == 0) {
            printf("OK Jail doors closed.");
            fflush(stdout);
        } else {
            printf("ERR Invalid command.\n");
            fflush(stdout);
            return 1;
        }
    } else {
        printf("ERR Authentication failed.\n");
        fflush(stdout);
        return 0;
    }
    return 0;
}

int main(int argc, char *argv[]) {
    int sockfd;
    int newsockfd;
    int port;
    int clientlen;
    char buffer[256];
    struct sockaddr_in server_addr;
    struct sockaddr_in client_addr;
    int n;
    int pid;
    int sockyes;
    sockyes = 1;
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0) {
        perror("Socket error");
        exit(1);
    }
    if (setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &sockyes, sizeof(int)) == -1) {
        perror("Setsockopt error");
        exit(1);
    }
    memset((char*)&server_addr, 0, sizeof(server_addr));
    port = 7411;
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(port);
    if (bind(sockfd, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
        perror("Bind error");
        exit(1);
    }
    listen(sockfd, 200);
    clientlen = sizeof(client_addr);
    while (1) {
        newsockfd = accept(sockfd, (struct sockaddr*)&client_addr, &clientlen);
        if (newsockfd < 0) {
            perror("Accept error");
            exit(1);
        }
        pid = fork();
        if (pid < 0) {
            perror("Fork error");
            exit(1);
        }
        if (pid == 0) {
            close(sockfd);
            exit(handle(newsockfd));
        } else {
            close(newsockfd);
        }
    }
}

```

#### code explanation

###### 1. **Global Variables and Function Declarations**

- `int debugmode;`: A global variable to control debugging output.
- Function declarations:
    - `int handle(int sock);` – Handles client requests on a socket.
    - `int auth(char *username, char *password);` – Authenticates the user.
###### 2 . **Authentication Function (`auth`)**

- Takes a `username` and `password` as arguments.
- Checks if the username is "admin" and if the password matches "1974jailbreak!".
- Returns `1` for successful authentication and `0` for failure.
- If `debugmode` is enabled, it prints the address of the `userpass` buffer.
###### 3. **Request Handling Function (`handle`)**

- **Setup:**
    
    - Sets `debugmode` to 0 and redirects stdout and stderr to the socket, so that responses are sent back to the client.
    - Prints a message prompting the client to send the USER command.
- **Processing Loop:**
    
    - Reads data from the socket into `buffer`.
    - Tokenizes the input buffer using newline (`\n`) as a delimiter.
    - Processes `USER`, `PASS`, and `DEBUG` commands:
        - `USER` command: Sets the `username` variable.
        - `PASS` command: Sets the `password` variable.
        - `DEBUG` command: Toggles debugging mode on/off.
    - Once both `USER` and `PASS` are received, the loop breaks.
- **Authentication and Command Handling:**
    
    - Calls the `auth` function to verify credentials.
    - If authentication is successful:
        - Reads another command from the socket.
        - Executes `OPEN` or `CLOSE` commands, or prints an error for invalid commands.
    - If authentication fails, prints an error message.

###### 4 . **Main Function (`main`)**

- **Setup:**
    
    - Creates a TCP socket with `socket()`.
    - Sets socket options with `setsockopt()`, specifically allowing address reuse.
    - Configures server address and port (7411).
    - Binds the socket to the specified address and port with `bind()`.
    - Listens for incoming connections with `listen()`.
- **Accepting Connections:**
    
    - Enters an infinite loop where it accepts incoming connections using `accept()`.
    - For each connection:
        - Forks a new process to handle the client using `fork()`.
        - In the child process, closes the listening socket and calls `handle()` to process the client request.
        - In the parent process, closes the connected socket and continues to accept new connections.

![](/Linux/Linux-Insane/Jail/Screenshots/buffer.png)

