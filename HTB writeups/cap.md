In this writeup we will talk about cap lab which is an easy lab in Hack the box

# nmap

```
╰─❯ nmap -sV  -sC 10.10.10.245
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-23 15:47 EEST
Stats: 0:00:34 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 75.15% done; ETC: 15:48 (0:00:11 remaining)
Stats: 0:00:55 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.06% done; ETC: 15:48 (0:00:00 remaining)
Nmap scan report for 10.10.10.245
Host is up (0.18s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    Gunicorn
|_http-title: Security Dashboard
|_http-server-header: gunicorn
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 60.32 seconds
```

http open then when we go to the web page we see a dashboard and theres pcap files associated to users

http://10.10.10.245/data/3

# IDOR

using idor you can make it http://10.10.10.245/data/0
which makes it to a new pcap file which can leak something

# analyzing PCAP files

![PCAP](/images/image.png)
theres a username and password if both ftp and ssh

# privilege escalation

when you enter as nathan user in the ssh you can see theres a file called py.py that changed the current linux user to 0 which is admin but running it gets you permission denied.

but its a python file! you can run python3 binary. try it . success and pwned
