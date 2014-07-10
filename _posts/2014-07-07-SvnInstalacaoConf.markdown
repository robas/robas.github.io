---
layout: post
title:  "Subversion: Instalação e Configuração Básica"
date:   2014-07-07 20:09:15
categories: serverdeploy
tags: controle de versão, versionamento, svn
---

Subversion
==========

# Download e Instalação SVN
apt-get update
apt-get install subversion

# Configuração SVN

## xinetd.conf
Se o internet super service não estiver instalado:
apt-get install xinetd

O arquivo /etc/xinetd.conf contém a seguinte cláusula:
"includedir /etc/xinetd.d"

Criamos o arquivo /etc/xinetd.d/svn, com o seguinte conteúdo:

{% highlight bash %}
# default: on
# description: Subversion server for the svn protocol
service svn
{
  disabled        = no
  port            = 3690
  socket_type     = stream
  protocol        = tcp
  wait            = no
  user            = subversion
  server          = /usr/bin/svnserve
  server_args     = -i -r /home/subversion/repositories
}
{% endhighlight %}
Note que na opção "server_args" indicamos o repositório com a opção "-r".. no caso acima coloquei a path "/home/subversion/repositories".
Note também que especificamos o usuário "subversion", que deve ser criado (> useradd subversion).

Com as novas configurações, reiniciamos o serviço xinetd:

> service xinetd restart

## Criação de um repositório
Com o usuário subversion, criamos um diretório de repositórios para conter nossos repositórios:
>mkdir /home/subversion/repositories
>mkdir /home/subversion/repositories/projeto1

>svnadmin create /home/subversion/repositories/projeto1

A execução do comando acima cria uma estrutura dentro do diretório /home/subversion/repositories/projeto1 para conter e gerenciar o versionamento de todos os arquivos. Vamos dar atenção especial aos seguintes arquivos:
>/home/subversion/repositories/projeto1/conf/svnserve.conf
>/home/subversion/repositories/projeto1/conf/passwd

## Autenticação e Permissão
No arquivo svnserve.conf o principal a ser configurado é:
>anon-access = none
>auth-access = write
>password-db = passwd
>realm = projeto1

anon-access regula qual o nível de acesso (read, write, none) de um usuário anônimo (não autenticado).
auth-access tem a mesma função explicada acima para usuários autenticados.
password-db é o nome do arquivo que contém os usuários e senhas para acesso ao repositório.
realm é o domínio de autenticação do repositório, se um repositório tem o mesmo domínio de autenticação de outro ele deve ter o mesmo arquivo _password-db_.

O arquivo de autenticação _passwd_ deve conter todos os usuários e senhas do repositório, seguindo a seguinte sintaxe:
>[users]
>usuário1 = senha1
>usuários2 = senha2
>...
>usuárion = senhan

## Pronto
A partir de agora já é possível conectar-se ao repositório, fazer uma importação de dados, checkout, etc:
> svn checkout svn://192.168.0.X/projeto1