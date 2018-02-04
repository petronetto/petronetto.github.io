---
layout: post
title: "Criando uma aplicação Laravel com Docker"
date: 2017-05-08 01:46
categories: laravel docker php
tags: laravel lumen docker php nginx
comments: true
image: /assets/images/articles/docker-laravel.png
---

Nesse artigo vou fazer uma breve introdução de como “Dockerizar” aplicação Laravel/PHP.

# Introdução

## Laravel
O que falar desse framework que conheço a pouco tempo e já considero pacas? Laravel é fácil de usar, muito bem documentado, tem a maior e mais ativa comunidade do PHP, e além disso tem uma sintaxe expressiva e elegante. O Laravel tenta tirar a dor do desenvolvimento facilitando tarefas comuns usadas na maioria dos projetos da web, como autenticação, roteamento, sessões e cache.

## Docker
Docker é uma tecnologia de virtualização, que elimina os aquela velha desculpa de “na minha máquina funciona”. Docker hoje é adotado em praticamente todas as grades empresas. Além disso, o Docker também é:

- Mais rápido e consome menos de recursos do que as VM’s tradicionais.
- Mais fácil de configurar e modificar.
- Fácil de reutilizar: você pode escolher uma imagem Docker existente e instalar quaisquer bibliotecas e pacotes ausentes (semelhante à herança de classe).
- Fácil de compartilhar: você pode enviar suas imagens para o Docker Hub e baixá-las para outras máquinas, de forma semelhante com seu código código Git (se você não usa git por favor saia dessa página agora, não queremos pessoas estranhas como você por aqui).

Se você ainda não usa Docker, apenas um conselho: USE!


### Instalação do Docker
Não vou estrar no mérito de como instalar, o Docker tem uma documentação excelente e tutoriais pra isso não faltam: (resumindo: se vira!)

- [Instalar no Mac](https://docs.docker.com/docker-for-mac/)
- [Instalar no Linux](https://docs.docker.com/engine/installation/linux/)
- [Instalar no Windows](https://docs.docker.com/docker-for-windows/)

### Mãos à obra… aliás… teclado
Para criar um container você precisa de um arquivo `Dockerfile`, que nada mais é que um arquivo de texto comum, que vai executar os comandos no Docker.

Crie um arquivo `Dockerfile` com o seguinte conteúdo:

{% gist 5676942c1a20d59b4c345f4b5211c8fb %}

### Arquivos de configurações
Calma ai jovem gafanhoto, ainda não acabou, tem muito código pela frente ainda.…  
Temos que criar agora os arquivos de configuração do Nginx e do Supervisor.

### Nginx
Crie um arquivo chamando `nginx.conf` com o conteúdo abaixo:
>> Note que essa é uma configuração **BÁSICA** do Nginx, não entrarei em detalhes dela. Procure na documentação oficial ou em outros blogs sobre esse assunto.
{% gist c5c19ca8ad443ad31feeceeff8751b0b %}

### Supervisor
Crie agora o aquivo de configuração do Supervisor. Ele será responsável por monitorar e gerenciar os processo do PHP e do Nginx, e caso algum dos processos morra ele irá reiniciá-los.  
Crie o arquivo `supervisord.conf`, com o conteúdo abaixo:

>> Novamente não entrarei no mérito dessas configurações, consulte a documentação oficial para mais detalhes: [supervisord.org](http://supervisord.org/)
{% gist 8bb69c3e3d70220058010ad5a109fea2 %}

## Fazendo a build do container
Bom, tudo configurado agora vamos fazer a build do nosso container, mas antes, faça seu cadastro no [Docker Hub](https://hub.docker.com/), pois vamos fazer o push do nosso container.

>> Dica do sucesso: use o mesmo nome do usuário do GitHub, não é necessário, mas vai por mim…


Para fazer a build nós vamos usar o comando `build`, seguido da tag `-t usuario/container`. A tag do container, é basicamente o seu nome de **usuário no Docker Hub / nome do container**, e por fim a localização do `Dockerfile`, partindo do pressuposto que está no mesmo diretório, iremos usar o `.`, que como você sabe (ou ao menos deveria saber), indica o diretório atual. O meu ficaria assim: 

`docker build -t petronetto/docker-laravel . ` 

Se tudo correu bem, ao usar o comando `docker images` você verá o container do Alpine e o seu container recém criado.  

## Push para Docker Hub
Uma vez que o container está “buildado”, você pode fazer o push para o Docker Hub com o comando: 
`docker push usuario/nome-do-container:tag`
A tag é optional, caso ela não seja informada por padrão a tag será `latest`.

No caso do nosso exemplo seria:  
`docker push petronetto/docker-laravel`


## Criando um projeto Laravel
Agora vamos finalmente criar um projeto com nosso querido *Laravel*.  
Lembra que instalamos o `Composer` no nosso container? Pois bem, provavelmente você já tem ele instalado localmente, mas para fins didáticos vamos criar um projeto usando o container que acabamos de criar:
{% highlight sh %}
docker run -it --rm \
    -v $(pwd):/app petronetto/docker-laravel \
    composer create-project laravel/laravel app
{% endhighlight %}

>> **Obs. 1**: Se você está usando o Windows <s>você deveria ser açoitado</s> e não tem o Git bash instalado, o `$(pwd)` não irá funcionar. Instale o `git bash`, `cmder`, `cygwin` ou melhor: **ainda instale um SO descente**.

>> **Obs. 2**: Não esqueça de alterar o `petronetto/docker-laravel` pelo nome do container que você criou.

Explicando o comando acima:  

- `it`: exibir o terminal iterativo, ou seja, o seu terminal exibirá as mensagens que forem exibidas no container.
- `rm`: remove o container após a execução do comando. Pode parecer inútil, mas depois de um tempo, você tira uma série de coisas da sua máquina, como Nginx, banco de dados e tal. Como você estará rodando no Docker, não faz sentido ter isso instalado localmente, sendo assim, quando você necessitar pode usar os recursos de um do seus container, como estamos fazendo agora, será muito útil.
- `v $(pwd):/app`: está fazendo um bind do volume do container Docker onde está a pasta `/app`, para diretório atual `$(pwd)`. Isso signigica, que qualquer alteração dentro da pata `/app` do container, vai refletir na sua pasta local.

Os demais comando são o nome do container em questão e o comando que você quer executar dentro dele. Mais pra adiante falarei mais detalhes sobre isso.

Nesse momento você vai notar que foi criada a pasta `app` no seu diretório local com um projeto Laravel novinho em folha.

### Colocando tudo pra funcionar
Agora vamos colocar tudo pra funcionar, seu o seguinte comando:
{% highlight sh %}
docker run -p 8080:80 \
    -v $(pwd)/app:/app \
    --name webserver \
    -d petronetto/docker-laravel
{% endhighlight %}

>> Novamente lembre-se de alterar o nome do seu container.

Nesse commando:

- `-p 8080:80` é para fazer o bind da sua porta 8080 com a porta 80 do container, sendo a ordem: `local:container`.
- `-v $(pwd)/app:/app` como expliquei anteriormente, faz o bind da pasta app que foi criada no seu diretório com a `/app` dentro do container. Todas alteração feitas no diretório local refletirá no do container e vise-versa.
- `--name webserver` é bem óbvio esse não é? É o nome da criança.
- `-d` é para rodar em modo *daemon*, ou seja, ele vai subir o container e liberar seu terminal, sem esse comando seu terminal ficará preso como quando você executa `php artisan serve`.


Agora acesse http://localhost:8080 e você verá a página inicial do Laravel.

## Docker Compose
Notou que tem uma penca de parâmetros para ficar passando? Seria sacal ter que decorar todos esses parâmetros e digitar todos esses comandos intermináveis no terminal, mas para nossa alegria existe o `Docker Compose`. Esse carinha vai abstrair boa parte disso em um arquivo `.yml`.  

Então bora lá, instale o [Docker Compose`](https://docs.docker.com/compose/install/).

Após instalado, crie um arquivo `docker-compose.yml`.

{% highlight sh %}
version: '3'
services:
  webserver:
    container_name: webserver
    image: petronetto/docker-laravel # ATENÇÃO!!! Aqui é o nome da SUA imagem
    volumes:
      - ./app:/app
    links:
      - database
    ports:
      - 8080:80
  database:
    container_name: database
    restart: always
    image: postgres:alpine
    ports:
      - 5432:5432
    environment:
      POSTGRES_DB: homestead
      POSTGRES_USER: homestead
      POSTGRES_PASSWORD: secret
    volumes:
      - ./database:/var/lib/postgresql
volumes:
   data:
      driver: local
{% endhighlight %}

>> Caso você queira fazer build direto pelo docker-compose, também é possível, altere a linha `image: petronetto/docker-laravel` para `build: .`.

Observe que estavamos usando o `Postgres` como banco de dados, pois ele tem uma versão do Alpine, que é uma distro Linux bem leve e pequena.  Caso queira usar o `MySQL` basta alterar o nome da imagem e as variáveis em `environment`. Mais detalhes [aqui](https://hub.docker.com/_/mysql/).

Agora altere o seu `.env`:
{% highlight sh %}
DB_CONNECTION=pgsql
DB_HOST=database
DB_PORT=5432
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
{% endhighlight %}

Perceba agora que em `DB_HOST` você vai usar o mesmo nome que informou no `docker-compose.yml`, nesse caso `database`.

Antes de rodar o comando para subir o docker-compose, remova o container que subimos anteriormente, pois caso contrário, você terá um erro pois ambos tem o mesmo nome e estão rodando na mesma porta. 
Remoca o container com `docker rm webserver -f`.

Agora suba os containers do docker-compose com `docker-compose up -d` e acesse novamente http://localhost:8080 e se tudo correu bem tela inicial do Laravel ainda estará lá.


### Gerando Auth scaffold
Agora vamos scaffold que o Laravel fornece:  
`docker exec -it webserver php artisan make:auth`  

Finalmente rode as migrations:  

`docker exec -it webserver php artisan migrate`  

Observe que agora usando o comando `exec` passando chamando o container pelo nome `webserver`, não por acaso, o mesmo nome que demos no arquivo do compose.  

Note também que como o PHP não está mais rodando na sua máquina, sempre que você precisar usar comando como migrations ou qualquer um que tenha interação com outros containers, você precisará executar esse comando através do container.  

Acesse mais uma vez http://localhost:8080 e se tudo correu bem você terá a tela inicial do Laravel pronta para o cadastro de usuários.


## Alguns comando úteis
No decorrer, pode ser que ocorra algum erro e você precise remover ou parar o container, então ai vai alguns comandos marotos: 

`docker images`: lista todos as imagens. 
`docker ps`: lista todos os container em execução. 
`docker ps -a`: lista todos os containers. 
`docker stop <id_do_container>`: preciso exlicar? 
`docker restart <id_do_container>`: preciso exlicar? 
`docker rm <id_do_container>`: remove um container. 
`docker rmi <id_do_image>`: remove uma imagem. Use o `-f` para forçar caso necessário.  

Também é possível combinar comandos para fazer umas coisas mais marotas ainda:

`docker stop $(docker ps -aq)`: o comando `stop` pede um id como entrada, o comando `docker ps -aq` retorna tos os ids, sendo assim você consegue para todos os containers. O mesmo vale para `restart`.  
`docker rm $(docker ps -aq)`: remove todos os containers.  
`docker rmi $(docker images -aq)`: remove todos as imagens. 


## Problemas comuns
Em distribuições linux, por ter um nível de acesso um pouco mais restrito que no Mac e no <s>RUIMdows</s> Windows, você pode precisar dar permissões a algumas pastas, facilmente resolvido com:  
`sudo chmod -R o+rw app/bootstrap app/storage` 
Se isso não resolver tente: `sudo chown -R $USER:$USER $(pwd)`.  

Muitos problemas também podem ser resolvidos apenas removendo a imagem ou reiniciando o container.


## Vamos falar sobre produção...
Como está dito no início do post, isso é uma *introdução*, para o bem e saúde de você e das pessoas que dependem do suas aplicações, eu espero de coração que você não seja o tipo de pessoa que lê um post introdutório e coloca isso em produção... Mas Caso você queira ter uma ideia de como vai funcionar em um ambiente de prod, bom isso é simples, mas não tenho como cobrir tudo aqui, principalmente porque existem várias formas de colocar um container Docker em produção e cada cloud provider tem suas particularidades. AWS, Digital Ocean, Heroku, etc.. Cada um terá uma vai ter algo diferente, mas basicamente, você pode seguir os mesmos passos ensinados aqui e entender o que foi feito você não deve ter dicifuldades. 

Um outro *disclaimer* importante, é: como eu disse, esse container é apenas para dar uma introdução, então muitas coisas não foram explicadas, e também não vá logo usando esse container num ambiente produtivo, aprenda um pouco mais sobre o Docker e no tempo certo aplique o que aprendeu em produção.


## Finalizando final finalmente
[Aqui](https://github.com/petronetto/laravel-docker) eu tenho basicamente tudo isso que foi ensinado aqui, é uma container mais "production ready". Dá uma conferida no [meu GitHub](https://github.com/petronetto) e lá vão ter vários outros containers interessantes que uso para facilitar meu dia-a-dia.  

É isso ai… Por hoje é só pessoal!  
Qualquer dúvida <s>pesquisa no Google porra!</s> poste nos comentários.
