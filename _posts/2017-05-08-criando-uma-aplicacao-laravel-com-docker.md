---
layout: post
title: "Criado uma aplicação Laravel com Docker"
date: 2017-05-08 01:46
categories: laravel docker php
tags: laravel docker php
comments: true
image: /assets/images/articles/docker-laravel.png
---

Nesse artigo vou fazer uma breve introdução de como “Dockerizar” aplicação Laravel/PHP.

# Introdução

## Laravel
O que falar desse framework que conheço a pouco tempo e já considero pacas? Laravel é fácil de usar e muito bem documentado e tem a maior e mais ativa comunidade, além de uma sintaxe expressiva e elegante. O Laravel tenta tirar a dor do desenvolvimento facilitando tarefas comuns usadas na maioria dos projetos da web, como autenticação, roteamento, sessões e cache.

## Docker
Docker, por outro lado, é um método de virtualização que elimina os aquela velha desculpa de “na minha máquina funciona”. Além disso, o Docker também é:

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

{% highlight sh %}
# Usaremos o container do Alpine que é considerávelmente
# menor do Debian ou Ubuntu
FROM alpine:3.5

# Instalando os pacotes necessários
# Note que instalaremos o Nginx juntamente com o PHP.
# Na filosofia do Docker essa não é uma prática 
# muito recomendável em todos os caso, pois o container
# em geral, deve rodar apenas um processo
# mas como o server interno od PHP não é recomendável
# para produção usaremos o Nginx e para não ter 
# que criar outro container apenas para o servidor
# web, instalaremos os dois no mesmo container
# e o supervisor cuidará dos processos
RUN apk --update add --no-cache \
        nginx \
        curl \
        supervisor \
        php7 \
        php7-dom \
        php7-fpm \
        php7-mbstring \
        php7-mcrypt \
        php7-opcache \
        php7-pdo \
        php7-pdo_mysql \
        php7-pdo_pgsql \
        php7-pdo_sqlite \
        php7-xml \
        php7-phar \
        php7-openssl \
        php7-json \
        php7-curl \
        php7-ctype \
        php7-session
# Limpando o cache das instalações
# é sempre recomendável remover do 
# container tudo aquilo que não for mais 
# necessário após tudo configurado
# assim o container fica menor
RUN rm -Rf /var/cache/apk/*

# Aqui criamos um symlink para o PHP7 como php apenas
# pois caso contrário, será necessário chamar o php
# como php7, e isso pode causar problemas no composer
RUN ln -s /usr/bin/php7 /usr/bin/php

# Instalando composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer

# Configurando o Nginx
# Aqui copiamos nosso arquivo de configuração para dentro do container
# Note que ainda não criamos esse arquivo, criaremos mais à frente
COPY nginx.conf /etc/nginx/nginx.conf

# Arquivo de configuração do supervisor
# Idem ao Nginx, será criado mais adiante
COPY supervisord.conf /etc/supervisord.conf

# Criando o diretório onde ficará nossa aplicação
RUN mkdir -p /app

# Definindo o diretório app como nosso diretório de trabalho
WORKDIR /app

# Dando permissões para a pasta do projeto
RUN chmod -R 755 /app

# Expondo as portas
EXPOSE 80 443

# Finalmente... Iniciando tudo... Ufa...
CMD ["supervisord", "-c", "/etc/supervisord.conf"]
{% endhighlight %}

### Arquivos de configurações
Calma ai jovem gafanhoto, ainda não acabou, tem muito código pela frente ainda.…  
Temos que criar agora os arquivos de configuração do Nginx e do supervidor.

### Nginx
Crie um arquivo chamando `nginx.conf` com o conteúdo abaixo:
>> Note que essa é uma configuração **BÁSICA** do Nginx, não entrarei em detalhes dela. Procure na documentação oficial ou en outros blogs sobre esse assunto.
{% highlight sh %}
user nginx;
worker_processes 2;

error_log /var/log/nginx/error.log;

pid /run/nginx.pid;
daemon off;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    server {
        listen   80;
        listen   [::]:80 default ipv6only=on;

        root /app/public;
        index index.php;
        access_log  /var/log/nginx/access.log  main;

        location ~ \.php$ {
            try_files $uri /index.php =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass 0.0.0.0:9000;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param SCRIPT_NAME $fastcgi_script_name;
            fastcgi_index index.php;
            include fastcgi_params;
        }

        location ~ ^(.*)$ {
            try_files $uri $uri/ /index.php?p=$uri&$args;
        }
    }
}
{% endhighlight %}

### Supervisor
Crie agora o aquivo de configuração do supervisor. Ele será responsável por monitorar os processo do PHP e do Nginx.  
Crie o arquivo `supervisord.conf`, com o conteúdo abaixo:

>> Novamente não entrarei no mérito dessas configurações, consulte a documentação oficial para mais detalhes: [supervisord.org](http://supervisord.org/)

{% highlight sh %}
[supervisord]
logfile=/tmp/supervisord.log
logfile_maxbytes=5MB
pidfile=/tmp/supervisord.pid
nodaemon=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0

[program:nginx]
command=/usr/sbin/nginx
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stdout_events_enabled=true
stderr_events_enabled=true

[program:php-fpm7]
command=php-fpm7 -F -c /etc/php7/php.ini -y /etc/php7/php-fpm.conf
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stdout_events_enabled=true
stderr_events_enabled=true
{% endhighlight %}

## Fazendo a build do container
Bom, tudo configurado agora vamos fazer a build do nosso container, mas antes, faça seu cadastro no [Docker Hub](https://hub.docker.com/), pois vamos fazer o push do nosso container.

>> Dica do sucesso: use o mesmo nome do usuário do GitHub, não é necessário, mas vai por mim…


Para fazer a build nós vamos usar o comando `build`, seguido da tag `-t usuario/container`. A tag do container, é basicamente o seu nome de **usuário no Docker Hub / nome do container**, e por fim a localização do `Dockerfile`, partido do pressuposto que está no mesmo diretório, iremos usar o `.`, que como você sabe (ou ao menos deveria saber), indica o diretório atual. O meu ficaria assim: 

`docker build -t petronetto/docker-laravel .` 

Se tudo correu bem, ao usar o comando `docker images` você verá o container do Alpine e o seu container recém criado.

## Push para Docker Hub
Uma vez que o container está “buildado”, você pode fazer o push para o Docker Hub com o comando: `docker push usuario/nome-do-container:tag`, a tag é optional, caso ela não seja informada por padrão a tag será `latest`.

No caso nosso caso seria: `docker push petronetto/docker-laravel`.


## Criando um projeto Laravel
Agora vamos finalmente criar um projeto com nosso querido *Laravel*.  
Lembra que instalamos o `Composer` no nosso container? Pois bem, provavelmente você já tem ele instalado localmente, mas para fins didáticos vamos criar um projeto usando o container que acabamos de criar:
{% highlight sh %}
docker run -it --rm \
    -v $(pwd):/tmp petronetto/docker-laravel \
    composer create-project laravel/laravel app
{% endhighlight %}

>> **Obs. 1**: Se você está usando o Windows <s>você deveria ser açoitado</s> e não tem o Git bash instalado, o `$(pwd)` não irá funcionar. Instale o `git bash`, `cmder`, `cygwin` ou melhor: **ainda instale um SO descente**.

>> **Obs. 2**: Não esqueça de alterar o `petronetto/docker-laravel` pelo nome do container que você criou.

Explicando o comando acima:  

- `it`: de para exibir o terminal iterativo, ou seja, o seu terminal exibirá as mensagens que forem imprimidas no container.
- `rm`: remove o container após a execução do comando. Pode parecer inútil, mas depois de um tempo, você tira uma série de coisas da sua máquina, como Nginx, banco de dados e tal. Como você estará rodando no Docker, não faz sentido ter isso instalado localmente, sendo assim, quando você necessitar pode usar os recursos de um do seus container, como estamos fazendo agora.
- `v $(pwd):/tmp`: está fazendo um bind do diretório atual `$(pwd)` para a pasta `/tmp`.

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

- `-p 8080:80` é para fazer o bind da sua porta 8080 coma porta 80 do container.
- `-v $(pwd)/app:/app` faz o bind da pasta app que foi criada no seu diretório com a `/app` dentro do container. Todas alteração do feita no diretório local refletirá no do container e vise-versa.
- `--name webserver` é bem óbvio esse não é? É o nome da criança.
- `-d` é para rodar em modo *daemon*, ou seja, ele vai subir o container e liberar seu terminal, sem esse comando seu terminal ficará preso como quando você executa `php artisan serve`.


Agora acesse http://localhost:8080 e você verá a página inicial do Laravel.

## Docker Compose
Notou que tem uma penca de parâmetros para ficar passando? Seria sacal ter que decorar todos esses parâmetros e digitar todos esses comandos intermináveis no terminal, mas para nossa alegria existe o `Docker Compose`. Esse carinha vai abstrair boa parte disso em um arquivo `.yml`.  

Então bora lá, instale o [Docker Compose`](https://docs.docker.com/compose/install/).

Após instalado, crie um arquivo `docker-compose.yml`.

{% highlight sh %}
version: '2'
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
      - "5432:5432"
    environment:
      POSTGRES_PASSWORD: homestead
      POSTGRES_USER: homestead
      POSTGRES_DB: secret
    volumes:
      - ./data:/var/lib/postgresql
volumes:
   data:
      driver: local
{% endhighlight %}

Observe que estavamos usando o `Postgres` como banco de dados, pois ele tem uma versão do Alpine, que é uma distro Linux bem leve e pequena.  Caso queira usar o `MySQL` basta alterar o nome da imagem e as variáveis em `environment`. Mais detalhes [aqui](https://hub.docker.com/_/mysql/).

Agora altere o seu `.env`:
{% highlight sh %}
DB_CONNECTION=pgsql
DB_HOST=database
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
{% endhighlight %}

Perceba agora que em `DB_HOST` você vai usar o mesmo nome que informou no `docker-compose.yml`, nesse caso `database`.

Acesse novamente http://localhost:8080 e se tudo correu bem ela estará lá ainda.


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
Em distribuições linux, por ter um nível de acesso um pouco mais restrito que no macOS e no <s>RUIMdows</s> Windows, você pode precisar dar permissões a algumas pastas, facilmente resolvido com:  
`sudo chmod -R o+rw app/bootstrap app/storage` 
Se isso não resolver tente: `sudo chown -R $USER:$USER $(pwd)`.  

Muitos problemas também podem ser resolvidos apenas removendo a imagem ou reiniciando o container.


## Finalizando final finalmente
[Aqui](https://github.com/Petronetto/laravel-docker) eu tenho basicamente tudo isso que foi ensinado aqui, e também tem outra meia dúzia de containers.  

É isso ai… Por hoje é só pessoal!  
Qualquer dúvida <s>pesquisa no Google porra!</s> poste nos comentários.