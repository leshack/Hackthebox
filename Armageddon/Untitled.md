## Common Enumeration
### Namp

##### Code
```code
nmap -vv 10.10.10.233 > nmap.txt
```


##### Nmap result
```code
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-06 10:20 EDT
Initiating Ping Scan at 10:20
Scanning 10.10.10.233 [4 ports]
Completed Ping Scan at 10:20, 0.41s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 10:20
Completed Parallel DNS resolution of 1 host. at 10:20, 0.41s elapsed
Initiating SYN Stealth Scan at 10:20
Scanning 10.10.10.233 [1000 ports]
Discovered open port 22/tcp on 10.10.10.233
Discovered open port 80/tcp on 10.10.10.233
Completed SYN Stealth Scan at 10:20, 2.51s elapsed (1000 total ports)
Nmap scan report for 10.10.10.233
Host is up, received echo-reply ttl 63 (0.34s latency).
Scanned at 2021-07-06 10:20:21 EDT for 3s
Not shown: 998 closed ports
Reason: 998 resets
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 3.51 seconds
           Raw packets sent: 1005 (44.196KB) | Rcvd: 1002 (40.076KB)
```

*we can see port 80 and 22 are open*
performing the second nmap with alot of more information

##### Code
```code
nmap -sV -sC -A -oN nmap.txt 10.10.10.233
```

##### Screenshot 1 Nmap 
![[Pasted image 20210706103310.png]]

fro that nmap information we can see the server is running on Apache 2.4.6 CentOs

we then do a Searchsploit to the Apace verstion

##### Screenshot 2 searchsploit
![[Pasted image 20210706104507.png]]
 
 we then open the browers and try to see how the page looks
 
 ![[Pasted image 20210706104817.png]]
 
 
