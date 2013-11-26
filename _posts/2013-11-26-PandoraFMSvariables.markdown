---
layout: post
title:  "Pandora FMS: Definição de variáveis"
date:   2011-03-23 22:32:46
categories: monitoring
tags: PandoraFMS, monitoring
---

Pandora FMS: Definição de variáveis
==================================

Ando utilizando o [Pandora FMS](http://pandorafms.com/ "Pandora Flexible Monitoring System") em um projeto e, tentando monitorar uma base de dados Oracle, senti certa dificuldade em especificar os diferentes SID que cada banco pode ter, afinal de contas, até para se fazer um simples tnsping deve-se saber o SID.
O intuito é fazer um módulo de monitoração genérico o bastante para ser aplicado a qualquer Oracle.

Foi aí que percebi que o agente de monitoração não permite, até onde eu saiba, a definição de variáveis particulares para cada agente.

Adicionei 2 linhas ao código do agente Pandora para plataformas Linux, fazendo-o reconhecer a definição de variáveis através do .conf.
No arquivo pandora_agent, na função read_config, adicionei as seguintes linhas:

{% highlight html %}
} elsif ($line =~ /^\s*env_var\s+(\w+)\s*=\s*(.+)$/) {
 $ENV{$1} = $2;
{% endhighlight %}

Ficando, portanto, assim:

{% highlight html %}
} elsif ($line =~ /^\s*file_collection\s+(.+)$/) {
 my $collection = $1;

 # Prevent path traversal attacks
 if ($collection !~ m/(\.\.)|\//) {
 $Collections{$collection} = 0;
 }
 # Environment Variable BEGIN
 } elsif ($line =~ /^\s*env_var\s+(\w+)\s*=\s*(.+)$/) {
 $ENV{$1} = $2;
 # Environment Variable END
 # Configuration token
 } elsif ($line =~ /^\s*(\S+)\s+(.*)$/) {

 log_message ('setup', "$1 is $2");
 $Conf{$1} = $2;

 # Remove trailing spaces
 $Conf{$1} =~ s/\s*$//;
 }
{% endhighlight %}


Desta forma é possível definir variáveis com a seguinte sintaxe no .conf:

{% highlight html %}
env_var minhavar1 = “lalala”

env_var variavel2 = “lelele”
{% endhighlight %}
