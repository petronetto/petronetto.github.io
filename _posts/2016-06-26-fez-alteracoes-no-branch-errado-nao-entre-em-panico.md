---
layout: post
title: "Fez alterações no branch errado? Não entre em pânico"
date: 2016-06-26 18:00
categories: git
tags: git
comments: true
crosspost_to_medium: true
image: /assets/images/articles/git.jpg
---

Uma coisa muito comum para quem trabalha com git é fazer dezenas de alterações e em algum momento perceber que fez as alterações no branch “X” quando deveria ter feito no branch “Y”. Como nos ensina o tão sábio *Guia do Mochileiro das Galáxias*: **NÃO ENTRE EM PÂNICO!**

Já vi muitas pessoas resolvendo isso de várias formas diferentes desde as mais simples como copiar os arquivos alterados, descartar as alterações e sair colando tudo novamente no branch correto, até as coisas mais mirabolantes possíveis como copiar o diretório, fazer um um clone do origin em um novo diretório, criar ou entrar no branch correto rodar um `git diff --stat` no diretório que tirou o backup para ver o que foi modificado.

Apesar disso tudo funcionar, o git fornece formas muito mais simples de resolver isso, a forma mais simples é usando o git stash.

Muitas vezes, quando você está trabalhando em uma parte do seu projeto, as coisas estão em um estado confuso e você quer mudar de branch por um tempo para trabalhar em outra coisa. O problema é, você não quer fazer o commit de um trabalho incompleto somente para voltar a ele mais tarde. A resposta para esse problema é o comando git stash.
Fazer Stash é tirar o estado sujo do seu diretório de trabalho — isto é, seus arquivos modificados que estão sendo rastreados e mudanças na área de seleção — e o salva em uma pilha de modificações inacabadas que você pode voltar a qualquer momento.
Fonte

Vavmos supor que em seu projeto tenha dois branches: master e funcionalidade-x. Por algum vacilo, você comeceu a mexer no master, quando eu deveria mexer no funcionalidade-x.

Partindo do pressuposto que você ainda não deu commit, tudo que foi alterado está apenas no diretório local, sendo assim, basta seguir os passos abaixo para corrigir:

{% highlight sh %}
git stash    # salva suas alterações
git stash branch temp    # salvando suas alterações em um branch temporário
git checkout funcionalidade-x     # fazendo checkout no branch correto
git merge --no-ff temp     # fazendo merge com o branch temporário 
git branch -d temp      # apgando o branch temporário
{% endhighlight %}

Obs.: A opção `--no-ff` diz ao git que ele tem sempre que criar um commit, indicando que foi feito um merge, iIsso vai fazer com que seu histórico tenha mais commits, mas vai te salvar quando você descobrir que fez um merge no branch errado, porque você vai poder fazer o reset.