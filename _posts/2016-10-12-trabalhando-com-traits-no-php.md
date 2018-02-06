---
layout: post
title: "Trabalhando com Traits no PHP"
date: 2016-10-12 19:32
categories: php
tags: php
comments: true
image: /assets/images/articles/php-traits.jpg
---

Traits são mecanismos que ajudam (e muito) a reutilização de código, e servem perfeitamente para resolver o problema da falta de herança múltipla.

Suponhamos que você tenha duas ou mais classes que precisam usar um método/comportamento em comum, antes da versão 5.4 você faria algo do tipo:

Primeiro temos nossa classe Log, que serviria pra salvar mensagens de log (imagine uma classe completa, que faça algo de útil):

{% highlight php %}
<?php
class Log
{
    public function log($message)
    {
        // Salva $message em um log de alguma forma
    }
}
{% endhighlight %}

E agora você tem outras classes que fazem uso dessa funcionalidade (salvar logs), mas essas classes não podem (e nem deveriam) estender Log, então você faria algo do tipo:

{% highlight php %}
<?php
Class Usuario extends Model
{
    protected $Log;
    public function __construct()
    {
        $this->Log = new Log();
    }
    public function save()
    {
        // Salva o usuário de alguma forma
        // ...
        // Salva uma mensagem de log
        $this->Log->log('Usuário criado');
    }
}

class Carrinho extends Produto
{
    protected $Log;
    public function __construct()
    {
        $this->Log = new Log();
    }
    public function clear()
    {
        // Limpa o carrinho de alguma forma
        // ...
        // Salva uma mensagem de log
        $this->Log->log('Carrinho de compras limpo');
    }
}
{% endhighlight %}

Mais uma vez, o conteúdo ou métodos dessas classes não importa.. o que importa aqui é o trabalho que temos para poder usar o Log::log() para salvar mensagens de log.

Já a partir da versão 5.4, podemos transformar a classe Log numa Trait:

{% highlight php %}
<?php
trait Log
{
    public function log($message)
    {
        // Salva $message em um log de alguma forma
    }
}
{% endhighlight %}

E manter o comportamento das nossas classes, de forma bem mais simples:

{% highlight php %}
<?php
Class Usuario extends Model
{
    use Log;
    public function save()
    {
        // Salva o usuário de alguma forma
        // ...
        // Salva uma mensagem de log
        $this->log('Usuário criado');
    }
}
{% endhighlight %}

{% highlight php %}
<?php
Class Carrinho extends Produto
{
    use Log;
    public function clear()
    {
        // Limpa o carrinho de alguma forma
        // ...
        // Salva uma mensagem de log
        $this->log('Carrinho de compras limpo');
    }
}
{% endhighlight %}

Podemos usar o método log() diretamente (sem a necessidade de instanciar um objeto de Log) pois nossas classes adquiriram as características (métodos e atributos) de Log! 🙂

### Conclusão

Traits são de fato recursos bem interessantes e muitos frameworks estão se adaptando à esse poderoso recurso.

Documentação oficial: [php.net/traits](http://php.net/traits)

É isso... Por hoje é só pessoal.