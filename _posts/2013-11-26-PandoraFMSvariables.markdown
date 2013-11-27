---
layout: post
title:  "Pandora FMS: Definição de variáveis"
date:   2011-03-23 22:32:46
categories: monitoring
tags: PandoraFMS, monitoring
---

Pandora FMS: Definição de variáveis
==================================

O [Pandora FMS](http://pandorafms.com/ "Pandora Flexible Monitoring System") é uma excelente ferramenta de monitoração, abrangendo infraestrutura e serviços.
Outro dia, desenvolvendo a monitoração para uma base de dados Oracle, esbarrei em uma dificuldade:

Como especificar o SID de cada banco, dado que o agente de monitoração não permite a definição de variáveis?

O intuito seria um módulo de monitoração genérico o bastante para ser aplicado a qualquer Oracle, mas sem a possibilidade de definir variáveis ou obtê-las do ambiente fica complicado.
Felizmente o core do Pandora é Open Source.
Adicionei 2 linhas ao código do agente Pandora para plataformas Linux (perl), fazendo-o reconhecer a definição de variáveis através do .conf.
No arquivo pandora\_agent, na função read\_config, adicionei as seguintes linhas:

{% highlight perl %}
} elsif ($line =~ /^\s*env_var\s+(\w+)\s*=\s*(.+)$/) {
  $ENV{$1} = $2;
{% endhighlight %}

Ficando, portanto, assim:

{% highlight perl %}
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

{% highlight bash %}
env_var minhavar1 = “lalala”
env_var variavel2 = “lelele”
{% endhighlight %}

E voilà! Agora é possível acessar, a partir de qualquer monitor desenvolvido, quaisquer variáveis definidas no .conf.
