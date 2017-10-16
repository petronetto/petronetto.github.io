---
layout: post
title: "Assegurando a qualidade de código com SonarQube"
date: 2017-10-15 21:09
categories: ci
tags: docker quality sonarqube
comments: true
image: /assets/images/articles/quality.jpg
---

Nesse artigo vou dar uma breve introdução de como usar o SonarQube para garantir melhor qualidade do seu código.

# Introdução

Se você é dev minimamente preocupado com a qualidade dos softwares que você e sua equipe desenvolve, com toda certeza em algum momento você já pegou se questionando sobre a qualiade do software que você ou sua equipe desenvolveu ou está desenvolvendo.
Mesmo que existam vários approachs para lidar com isso, a revisão "humana" sempre tende a ser muito... como posso dizer... humana!
A alguns anos atrás isso era uma tarefa não muito trivial, mas felizmente hoje em dia é muito mais simples, devido  ao advendo dos softwares de *code quality*.


O Sonar é uma ferramenta de análise de qualidade de código baseado na web para projetos Java baseados em Maven. Abrange uma ampla área de pontos de verificação de qualidade de código que incluem: Arquitetura e Design, Complexidade, Duplicações, Regras de Codificação, Bugs Potenciais, Teste de Unidade, etc. O Sonar possui um rico conjunto de recursos como o que você obteria com diferentes ferramentas como Covertura, PMD, FindBugs, Check Styles combinados.
