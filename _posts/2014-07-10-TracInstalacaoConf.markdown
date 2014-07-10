---
layout: post
title:  "Trac: Instalação e Configuração Básica"
date:   2014-07-10 16:39:55
categories: serverdeploy
tags: bug tracking tool
---

Trac
====

A ordem de instalação das dependências, opcionais ou não, é muito importante para que tudo funcione corretamente.
# Python
O trac, e suas dependências, rodam sobre o python, se você não tem python na máquina nem sei o que está fazendo aqui... Dê logo um

>apt-get install python

# Internacionalização (Babel)
Se deseja ter a possibilidade de usar o trac com a tradução para a sua (e mais) línguas, é necessário instalar o Babel __ANTES__ de iniciar a instalação do trac.

>apt-get install python-babel

# Instalação
Graças ao apt-get as dependências são instaladas todas com esse simples fucking comando abaixo (e eu que sofri instalando sem o apt-get)

>apt-get install trac

# Configuração inicial
Antes de tudo é necessário criar um novo projeto para então podermos configurá-lo. Isto é feito através da criação de um _ambiente_, que é onde o Trac guarda informações como páginas wiki, tickets, relatórios, etc:
> trac-admin /caminho/para/projeto1 initenv

O comando acima nos perguntará:
* O nome do projeto: escolha o nome que quiser
* A conexão à base de dados: no nosso caso vamos utilizar o sqlite, que é a opção default, basta pressionar enter e ele criará o arquivo da base sqlite (/caminho/para/projeto1/db/trac.db.)

# Execução do servidor
O Trac pode ser executado como um servidor StandaAlone (já embutido nele) ou sobre um Apache. A idéia aqui, como não necessitamos de nada complexo, é executarmos o servidor no modo StandAlone.
A partir de agora já é possível executarmos o servidor com o seguinte comando e conectar pelo browser:

> tracd --port 8000 /path/to/myproject

Maaaaas, ao clicar em "Login" receberemos um erro, mas relaxe, isso é porque ainda não configuramos a parte de autenticação do Trac, então vamos lá.

# Autenticação
A autenticação utilizada pelo Trac depende do modo como ele é executado, no nosso caso (StandAlone) a configuração é feita através de um arquivo contendo usuários e senhas:
O arquivo deve seguir o seguinte formato:
_usuario:realm:senha_

Sendo que a _senha_ é, na verdade, um md5 do usuário + realm + senha original, exemplo:

usuário: robas
realm: empresaX
senha: robas

A entrada no arquivo de autenticação para os dados acima seria:
robas:empresaX:789f5680d4a56d9c85109827b2f29bab

Para obtermos a string "789f5680d4a56d9c85109827b2f29bab", podemos fazer:
>printf "robas:empresaX:robas" | md5sum

Repita para cada usuário desejado e monte o arquivo de autenticação.

Obs: Para facilitar a montagem do arquivo de autenticação é possível usar os seguintes comandos:
>user=usuario1
>realm=empresa
>password=senha1
>path_to_file=/caminho/para/arquivo/de/autenticação
>echo ${user}:${realm}:$(printf "${user}:${realm}:${password}" | md5sum - | sed -e 's/\s\+-//') >> ${path_to_file}

Depois basta alterar o valor das variáveis _user_ e _senha_ e executar o último comando, desta forma os novos usuários serão adicionados no arquivo de autenticação.

É necessário darmos permissão de administrador para algum usuário, entre os listados no arquivo de autenticação: 
> trac-admin /caminho/para/projeto1 permission add usuario1 TRAC_ADMIN

# Iniciar o servidor standalone com autenticação

> tracd --port 8000 --auth="NomeDoProjeto,/caminho/para/arquivo/de/autenticação,realm" /caminho/para/projeto

Exemplo:

> tracd --port 8000 --auth="projeto1,/home/trac/conf/.authdb,empresaX" /home/trac/projeto1

# Customização do projeto
Para customizar o logo no trac:
No arquivo /home/subversion/trac/projeto1/conf/trac.ini

>[header_logo]
>alt = SG2i
>height = 224
>link = http://192.168.1.53:8000/projeto1
>src = site/sg2i-logo.png
>width = 416

Na configuração acima o _src_ quando especificado o caminho relativo _site_ se refere ao diretório _htdocs_ dentro da estrutura do projeto no trac.

Deve-se também customizar as strings existentes na seção [project]

# Integração com Subversion
Para habilitar a integração com o subversion devemos incluir um repositório, realizar um sincronização inicial e configurar os hooks (triggers) de execução (sempre que um commit ocorrer no repositório, sincronizar o trac com o conteúdo alterado).

## Registrar repositórios no Trac
No arquivo trac.ini acrescentar as seguintes seções:

>[components]
>tracopt.versioncontrol.svn.* = enabled
>
>[repositories]
>cigerepo.dir = /home/subversion/repositories/projeto1
>cigerepo.description = "Repositório do projeto CIGE"
>cigerepo.type = svn
>cigerepo.url = svn://localhost/projeto1

## Sincronização inicial
Agora que acabamos de configurar o repositório _cigerepo_, vamos fazer yum resync manual para ver se está tudo correto:
> trac-admin caminho/para/projeto1 repository resync cigerepo

Ao entrar na parte de _Ver Código_ no menu superior da interface web do Trac deve ser possível visualizar o repositório _cigerepo_ e seus arquivos.

## Hooks
Devemos configurar também os seguintes triggers de execução:
* post-commit
* post-revprop-change

Temos modelos destes scripts no diretório _hooks_ do nosso subversion, a sugestão é copiar os templates já existentes, modificá-los e torná-los executáveis (chmod +x _arquivo_):

Arquivo _post-commit_:
>#!/bin/sh
>REPOS="$1"
>REV="$2"
>export PYTHON_EGG_CACHE="/path/to/dir"
>/usr/bin/trac-admin /home/subversion/trac/projeto1 changeset added "$REPOS" "$REV"


Arquivo _post-revprop-change_:
>#!/bin/sh
>REPOS="$1"
>REV="$2"
>export PYTHON_EGG_CACHE="/usr/local/lib/python2.7"
>/usr/bin/trac-admin /home/subversion/trac/projeto1 changeset added "$REPOS" "$REV"



# Trac as a service

Para cadastrar o trac como um serviço no debian e ser executado automaticamente na inicialização do sistema:

Gerar o seguinte arquivo "tracd" no diretório /etc/init.d:

{% highlight bash %}
#!/bin/bash
#
### BEGIN INIT INFO
# Provides:             trac
# Required-Start:       $network
# Required-Stop:        $network
# Default-Start:        2 3 4 5
# Default-Stop:         0 1 6
# Short-Description:    Trac standalone web server
# Description:          Standalone Webserver for Trac
### END INIT INFO

start() {
  tracd --port 8000 --auth="CIGE,/home/subversion/trac/cige/conf/.authdb,cige" /home/subversion/trac/cige &
}

stop() {
  ps -ef | grep "tracd" | grep -v grep | awk '{print $2}' | xargs kill
}

restart() {
  stop
  start
}                                                                                                                                                              
                                                                                                                                                               
case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  restart)
    restart
    ;;
  *)
    echo $"Usage: $0 {start|stop|restart}"
    exit 1
esac

exit $?
{% endhighlight %}

Tornálo executável:
>chmod +x tracd

E incluí-lo na inicialização do sistema:
>update-rc.d tracd defaults