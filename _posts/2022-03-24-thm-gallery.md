---
title: THM - Gallery
author: guialvesf
date: 2022-03-24 01:00:00 -0300
categories: [tryhackme, writeup]
tags: [tryhackme, writeup]
image: "/img/thm/gallery/Capa.png"
alt: "tryhackme"
---


<p align="center" width="100%">
    <img src="/img/thm/gallery/Capa.png"> 
</p>
<hr>

Primeiramente, vamos usar o Nmap para encontrar portas e serviços abertos no sistema.

![nmap](/img/thm/gallery/NmapOutput.png)

Como podemos ver há duas portas abertas no sistema que estão executando Apache: 80 e 8080, então acessando podemos ver que a porta 80 tem apenas a página padrão do Apache, enquanto a porta 8080 está rodando o CMS Simple Image Gallery System.

![cms](/img/thm/gallery/SimpleImageGallerySystemLogin.png)

Ao invés de procurar por algum exploit pronto, podemos tentar explorar o sistema por conta próprio. Apenas tentando um payload básico de injeção SQL podemos acessar a conta de Admin.

`' or 1=1 -- - `

![sqli](/img/thm/gallery/SimpleImageGallerySystemSQLi.png)

Agora podemos procurar alguma maneira de conseguir uma shell, e como podemos ver temos a permissão para fazer upload de arquivos para o servidor. A primeira coisa que podemos tentar é subir uma shell em PHP para recebermos pelo Netcat. Usei a shell do PentestMonkey

![phpshell](/img/thm/gallery/FirstSession.png)

Pronto, conseguimos uma conexão reversa, porém estamos logados como www-data então precisamos elevar os privilégios, e para isso chequei o código fonte da aplicação, onde encontrei as credenciais do banco de dados.

![initialize](/img/thm/gallery/Initialize.png)

Agora que temos a senha podemos acessar o banco.

![select](/img/thm/gallery/DatabaseSelected.png)

Acessando o banco conseguimos a senha de Admin para responder à terceira questão.

Porém se tentarmos ler a flag de usuário veremos que não temos permissões o suficiente

![home](/img/thm/gallery/MikeHome.png)

Precisaremos então escalar nosssos privilégios, e após analisar a máquina encontrei um diretório de backup

![backup](/img/thm/gallery/MikeBackup.png)

O diretório contém o arquivo .bash_history, onde contém o histórico de comandos executados pelo usuário, e junto a isso a senha usada pelo mesmo.

![password](/img/thm/gallery/MikePassword.png)

Pronto, conseguimos a flag de usuário, agora vamos atrás da conta root.

![userflag](/img/thm/gallery/UserFlag.png)

A primeira coisa que costumo fazer para a escalação de privilégios é checar quais comandos consigo executar como

![optfile](/img/thm/gallery/sudo-l.png)

Lendo o código fonte deste script podemos ver que temos a permissão de executar o Nano com privilégios root.

![nano](/img/thm/gallery/nano.png)

Usando o GTFObins encontrei a forma pela qual iremos fazer a escalação.

![nanogtfo](/img/thm/gallery/nanoGTFOBins.png)

Porém para isso precisamos estabilizar nossa shell. Para isso basta colocar a shell em background usando CTRL+Z e depois digitar
`stty raw -echo; fg`

![ctrlz](/img/thm/gallery/ctrlz.png)

Agora basta executar o script como root, escolher a opção do Nano e digitar os comandos.

![nanocommand](/img/thm/gallery/nanoCommand.png)

Pronto, encontramos a flag de root.

![root](/img/thm/gallery/root.png)
