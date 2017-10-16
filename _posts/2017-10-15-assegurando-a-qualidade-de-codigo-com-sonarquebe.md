---
layout: post
title: "Assegurando a qualidade de código com SonarQube"
date: 2017-10-15 21:09
categories: ci
tags: docker quality sonarqube
comments: true
image: /assets/images/articles/quality.jpg
---

Nesse artigo vou dar uma breve introdução de como usar o SonarQube para garantir uma melhor qualidade do seu código.


## Introdução  

Se você é um dev minimamente preocupado com a qualidade dos softwares que você e sua equipe desenvolvem, com toda certeza, em algum momento você já se pegou questionando-se sobre a qualidade do software que você ou sua equipe desenvolveram ou estão desenvolvendo.
Mesmo que existam várias técnicas e métodos para se lidar com isso, a revisão "humana" sempre tende a ser muito... como posso dizer... humana.  

A alguns anos era uma tarefa não muito trivial, mas felizmente hoje em dia é muito mais simples, devido ao advendo dos softwares de **code quality**, e uma das melhores e mais completas ferramentas desse tipo, na minha opnião é o [SonarQube](https://www.sonarqube.org)  



## O que é o SonarQube  

O SonarQube é uma ferramenta de análise de qualidade de código, open source, que abrange uma ampla área de pontos de verificação de qualidade de código que incluem: arquitetura e design, complexidade, Duplicações, regras de codificação, potenciais bugs, testes unitários, etc. O Sonar possui um rico conjunto de recursos que ajudam a manter a qualidade do seu código.

O SonarQube provê a capacidade de mostrar não apenas a saúde de uma aplicação, mas também de destacar os problemas recém introduzidos. Agindo como um Quality Gate, você pode corrigir, monitorar, e aprimorar a qualidade do código sistematicamente.

![SonarQube Dashboard]({{ "/assets/images/articles/sonarqube.png" | absolute_url }})



## Começando

Para usar o SonarQube existem várias opções, você pode instalá-lo localmente, num seridor ou até mesmo usar o serviço de cloud fornecido pela Sonar. Para esse tutorial, por motivos de <s>eu quero</s> comodidade, usaremos uma imagem Docker que criei para usar nos meus projetos, sendo assim, assegure que você tem o Docker instalado e funcionado.

Além do SonarQube, também iremos precisar do Sonnar Scanner, que nada mais é que um executável que irá realizar o scan do nosso código e enviá-lo para o sua aplicação do SonarQube. Como eu odeio ficar instalando milhões de ferramentas, felizmente, no container que usarei o Sonar Scanner já está instalado.



## Talk is cheap. Show me the code

Pelo terminal, acesse a pasta do projeto cujo o qual você deseja analizar e rode o seguinte comando:
{% highlight sh %}
docker run -p 9000:9000 --name sonarqube -d petronetto/sonarqube-alpine
{% endhighlight %}

Aguarde alguns instantes até que o serviço possar ser iniciado e acesse `http://localhost:9000`, se tudo correr bem você irá ver a tela abaixo:
![SonarQube About]({{ "/assets/images/articles/sonar-about.png" | absolute_url }})

Clique em login entre com as credenciais `admin`, senha `admin`. Ao logar irá surgir um pop-up, click em `Skip this tutorial`.

Pronto, agora você já tem um servidor do SonarQube funcional, pronto para receber os scans do seu código. Para isso você tem duas opções, baixar e instalar o [Sonnar Scanner](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner) localmente, ou usar o que está disponível no container que está rodando o SonarQube. Usaremos a segunda opção, mas antes, vá até `http://localhost:9000/updatecenter/` e verifique se o plugin para sua liguagem está instalado. O SonarQube por padrão já vem com C#, Python, Java PHP, JavaScript e outros. Caso sua linguagem não esteja na listagem do Update Center, dê uma Googlada e provavelmente vai achar uma modo de instalar ;)

Agora vamos fazer a análise do nosso código, basta rodar o comando:
{% highlight sh %}
docker run --rm -it \
    -v $(pwd):/data \
    --link sonarqube \
    petronetto/sonarqube-alpine sonar-runner \
    -Dsonar.projectKey="MyKey" \
    -Dsonar.projectName="My Project" \
    -Dsonar.projectBaseDir=/data \
    -Dsonar.sources=/data \
    -Dsonar.host.url=http://sonarqube:9000
{% endhighlight %}

Aguarde até o scan terminar, dependendo da quantidade de código isso pode demorar bastante, e se você não entendeu bulhufas do que aconteceu ai em cima, calma! Eu explico. 
No comando acima, basicamente criamos um container que foi executado apenas para rodar o comando `sonar-runner [...params]`, para isso foi usado o parâmetro do Docker run `--rm`, que remove o container assim que o comando é finalizado. 
Também mapeamos o diretório atual, onde está nosso código, para uma pasta dentro do container, `/data`, onde o código será analizado pelo Sonar Scanner, usando `-v $(pwd):/data`.
Logo após, criamos um link apontando para o container que está rodando SonarQube e irá receber os dados do scan, usando `--link sonarqube`.
Por fim, apontamos a imagem que queríamos rodar, no caso `petronetto/sonarqube-alpine`, seguido do comando que seria executado dentro do container, `sonar-runner [...params]`
> Observe que no parâmetro `-Dsonar.host.url=http://sonarqube:9000`, onde `sonarqube` é o nome que demos ao nosso container. Caso você tenha dado um nome diferente altere esse parâmentro para o nome que você deu.

Bom, se tudo correu bem, ao acessar o seu dashboard você deverá ver uma tela similar a tela abaixo:
![Seus projetos]({{ "/assets/images/articles/sonar-projects.png" | absolute_url }})



## Finalizando

Pronto, agora você tem uma análise completa do seu código, e se você chegou até aqui, deve ter percebido que é bem fácil subir um server com o SonarQube bem como integrá-lo no seu pipeline de CI.

Você também pode dar uma checada no repositório onde está o código desse container que usamos: [https://github.com/petronetto/sonarqube-alpine](https://github.com/petronetto/sonarqube-alpine). Lá tem alguns exemplos mais avançados e tal.

Caso tenha alguma dúvida, vale a pena dar uma olhada na [documentação do SonarQube](https://docs.sonarqube.org/display/SONAR/Documentation).


Até a próxima!


