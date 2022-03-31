---
title: THM - Gallery
author: guialvesf
date: 2022-03-24 01:00:00 -0300
categories: [tryhackme, writeup]
tags: [tryhackme, writeup]
image: "/img/thm/gallery/Capa.png"
alt: "tryhackme"
---

![banner](/img/thm/gallery/Capa.png)
</p>
<hr>

First, we can use Nmap to find open ports and services running

![nmap](/img/thm/gallery/NmapOutput.png)

As we can see, there are two open ports running apache: 80 and 8080, then accessing it we can see that 80 port is running default apache page, but 8080 port is running a Simple Image Gallery System

![cms](/img/thm/gallery/SimpleImageGallerySystemLogin.png)

Instead of look for an exploit, we can try to exploit it by ourselves. Trying a simple SQLi payload we can get access to admin account.

`' or 1=1 -- - `

![sqli](/img/thm/gallery/SimpleImageGallerySystemSQLi.png)

Ok, now we need to look for a way to get a shell. As we can see, we can upload images and files to server. Can we try to upload a reverse shell and receive it via netcat. To do this we'll use PentestMonkey's php-reverse-shell.php

![phpshell](/img/thm/gallery/FirstSession.png)

Nice, we got a shell, so let's look for admin's hash. To do this we have a php file with database connection, so oppening it we can see the database's password.

![initialize](/img/thm/gallery/Initialize.png)

Ok, so now let's access the database.

![select](/img/thm/gallery/DatabaseSelected.png)

After accessed, we can see `gallery_db` which probably contains admin's password to answer the third question.

Now if we try to read user flag we'll see that we don't have permissions.

![home](/img/thm/gallery/MikeHome.png)

So now we need to escalate our privileges. After analyzing the machine, I found a backup file that we can read.

![backup](/img/thm/gallery/MikeBackup.png)

Openning this directory we can read Mike's password in his .bash_history file

![password](/img/thm/gallery/MikePassword.png)

Now we must access his account to read user flag and then escalate (again) our privileges, but now we're going to escalate it to root user.

![userflag](/img/thm/gallery/UserFlag.png)

The first thing to do with a user session is check our permissions with root. So typing sudo -l we can see a script file that we can run as root.

![optfile](/img/thm/gallery/sudo-l.png)

We can read this script's source code, and check that we can run nano (as root).

![nano](/img/thm/gallery/nano.png)

Then looking for something related, we found the GTFObins, which we'll use to escalate our privileges to root user.

![nanogtfo](/img/thm/gallery/nanoGTFOBins.png)

But we can't do it without a established shell. So we'll put our shell in background pressing `CTRL+Z` and then typing 
`stty raw -echo; fg`
![ctrlz](/img/thm/gallery/ctrlz.png)

Nice, we can now execute the script as root, choose Nano to run, and then type a command to get a root shell.

![nanocommand](/img/thm/gallery/nanoCommand.png)

So now we can read root flag

![root](/img/thm/gallery/root.png)
