---
title: THM - Mustacchio
author: guialvesf
date: 2022-03-30 16:50 -0300
categories: [tryhackme, writeup]
tags: [tryhackme, writeup]
image: "/img/thm/mustacchio/banner.png"
alt: "tryhackme"
---

![banner](/img/thm/mustacchio/banner.png) 
<hr>

# Recon

Vamos começar com o scan

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

Encontrei três serviços rodando. Apache, Nginx e SSH, então vamos começar pela aplicação web

![webapp](/img/thm/mustacchio/webpage.png)

# User

Analisando a página inicial não foi encontrado nada de muito interessante, porém olhando o código fonte da página vi que um script estava sendo executado, a partir do diretório custom

![sourcecode](/img/thm/mustacchio/viewsource.png)

Acessando esse diretório encontrei o arquivo `users.bak`. Bainxando o mesmo nós podemos analisar melhor.

![adminhash](/img/thm/mustacchio/adminhash.png)

Ok, encontramos duas informações: usuário `admin` e a hash da senha, então podemos usar o [crackstation](https://crackstation.net/) para quebrá-la

![crackstation](/img/thm/mustacchio/crackstation.png)

* Usuário: admin
* Senha: bulldog19

Agora consigo tentar usar essas credenciais no serviço que está rodando na porta 8765.

![adminpanel](/img/thm/mustacchio/adminpanel.png)

Acessando o código fonte da página encontrei um comentário deixado por um usuário, nos informando que o user Barry pode acessar o SSH usando uma chave. 
Há também outro arquivo .bak que podemos baixar no diretório auth.

![sourcecode2](/img/thm/mustacchio/viewsource2.png)

O arquivo trata-se de um XML, então podemos tentar explorar uma falha de XXE na aplicação.

![dontforget](/img/thm/mustacchio/dontforget.png)

A princípio testei apenas um XML para ver o comportamento do site.

![xmltest](/img/thm/mustacchio/xmlteste.png)

Então usando um payload de XXE nós conseguimos acessar o arquivo `/etc/passwd`

![passwd](/img/thm/mustacchio/passwd.png)

Agora podemos roubar a chave SSH privada do usuário Barry. Para isso vamos usar a falha de Local File Inclusion para ler o arquivo.

![id_rsa](/img/thm/mustacchio/id_rsa.png)

Infelizmente não conseguimos acessar a conta apenas com a chave, precisamos da senha do id_rsa.

![ssh_req](/img/thm/mustacchio/id_rsa_pass_req.png)

Então podemos usar o John The Ripper para fazer um bruteforce.

![john](/img/thm/mustacchio/johntheripper.png)

E assim conseguimos quebrar a senha: `urieljames`

![ssh](/img/thm/mustacchio/ssh.png)

Agora sim podemos fazer acesso usando o SSH com a chave e senha conseguidos.

![user](/img/thm/mustacchio/user.png)

# Root

Ao iniciar a escalação de privilégios, notei que há dois usuários e dois diretórios home.

![home](/img/thm/mustacchio/joe.png)

No diretório do usuário Joe tem um arquivo, e usando novamente o `file` e `strings` conseguimos ver o `tail` sendo chamado.

![log_file](/img/thm/mustacchio/strings.png)

Para fazer a exploração basta criar um arquivo chamado `tail` que execute o bash e alterar a variável de ambiente PATH.

![tail](/img/thm/mustacchio/tail.png)

Pronto, agora temos acesso root ao sistema e conseguimos a última flag.

![root](/img/thm/mustacchio/root.png)
