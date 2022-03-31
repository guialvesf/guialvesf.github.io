---
title: THM - Mustacchio
author: guialvesf
date: 2022-03-30 16:50 -0300
categories: [tryhackme, writeup]
tags: [tryhackme, writeup]
image: "/img/thm/mustacchio/banner.png"
alt: "tryhackme"
---

<p align="center" width="100%">
    <img src="/img/thm/mustacchio/banner.png"> 
</p>
<hr>

# Recon

Let's start with a Nmap scan

```
gui@gui:~$ nmap -sC -sV -oA nmap -p- -v 10.10.253.75
Nmap scan report for 10.10.253.75
Host is up (0.24s latency).
Not shown: 65532 filtered ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 58:1b:0c:0f:fa:cf:05:be:4c:c0:7a:f1:f1:88:61:1c (RSA)
|   256 3c:fc:e8:a3:7e:03:9a:30:2c:77:e0:0a:1c:e4:52:e6 (ECDSA)
|_  256 9d:59:c6:c7:79:c5:54:c4:1d:aa:e4:d1:84:71:01:92 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS GET HEAD
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Mustacchio | Home
8765/tcp open  http    nginx 1.10.3 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Mustacchio | Login
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
* -sC - Default Nmap scripts
* -sV - Services version
* -oA - Save output to a file
* -p- - All 65535 ports
* -v - Verbose mode

Ok, we found three services running on this machine. Apache, Nginx and SSH, so let's start with Apache.

![webapp](/img/thm/mustacchio/webpage.png)

# User

Analyzing the page I didn't find anything, so I checked the page's source code, and discovered a interesting thing: there's a script being called 
from `custom` directory

![sourcecode](/img/thm/mustacchio/viewsource.png)

So acessing this directory and then the js subdirectory I found a `users.bak` file. So by downloading we'll can analyze it better.

![adminhash](/img/thm/mustacchio/adminhash.png)

Ok, we have two new informations: `admin` user and its hash password. We can now use [crackstation](https://crackstation.net/) to crack it.

![crackstation](/img/thm/mustacchio/crackstation.png)

* User: admin
* Pass: bulldog19

Cool, now we can try to use these account credentials on service running on 8765 port.

![adminpanel](/img/thm/mustacchio/adminpanel.png)

Accessing and viewing the source code again, can we see that a SSH private key may be our attack point, and we can too download a new file.

![sourcecode2](/img/thm/mustacchio/viewsource2.png)

Analyzing this file, can I see that the admin panel its something about XML, so can we try a XXE. 

![dontforget](/img/thm/mustacchio/dontforget.png)

So now can we test it.

![xmltest](/img/thm/mustacchio/xmlteste.png)

Trying a basic XXE payload, we can access the `/etc/passwd` file.

![passwd](/img/thm/mustacchio/passwd.png)

Ok, now we'll need to steal Barry's ssh private key. For this we'll use that file inclusion to read the file, and then use it to access the machine. I used the source code to copy, because of the formatting.

![id_rsa](/img/thm/mustacchio/id_rsa.png)

NICE, now can we use this ssh private key to guarantee access? No :(

![ssh_req](/img/thm/mustacchio/id_rsa_pass_req.png)

Now we need to crack another password, but this time I'll use the famous John The Ripper to do it.

![john](/img/thm/mustacchio/johntheripper.png)

Nice, another password cracked, so now we can access.

![ssh](/img/thm/mustacchio/ssh.png)

Now let's just read user flag

![user](/img/thm/mustacchio/user.png)

# Root

For this privilege escalation, I noticed it has two users and two home directories

![home](/img/thm/mustacchio/joe.png)

And Joe's home directory has a file, and analyzing it with `file` and `strings` I found something a little interesting: tail is being invoked.

![log_file](/img/thm/mustacchio/strings.png)
