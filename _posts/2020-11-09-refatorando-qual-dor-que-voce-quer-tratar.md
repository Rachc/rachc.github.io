---
layout: post
title:  Refatorando - Qual dor você quer tratar?
date:   2019-11-09 10:40:00 -0300
image:  refactoring00.jpg
tags:   Refactor
---
Quando começamos a estudar técnicas de refatoração, nos deparamos com diversos (anti)padrões que podemos encontrar no nosso código.

Porém muitos desses padrões possuem nomes que não são muito claros e que, a primeira vista, fica difícil entender o que eles são.

Tendo essa dificuldade, resolvi criar um guia para identificar quais padrões são esses de acordo com a dor que sentimos ao ver o código.

## 1. Código que cresceram em uma proporção muito grande e são difíceis de trabalhar (Bloaters)
1. **Métodos muito longos** = Long Method
2. **Classes muito grandes** = Large class
3. **Longa lista de parâmetros** = Long parameter list
4. **Agrupamento de dados repetidos** = Data Clump
5. **Excesso de primitivo - muitas constantes - ou primitivos que poderiam ser objetos** = Primitive Obsession

## 2. Problemas derivados de um mal uso de orientação a objeto (Object-Orientation Abusers)
1. **Muitos ifs / switch case** = Switch Statement
2. **Variável temporária** = Temporary Field
3. **Duas classes com mesma função mais nomes diferentes** = Alternative Classes with different Interfaces
4. **Minha classe filha usa bem pouca coisa da classe pai** = Refused Bequest

## 3.Problemas que fazem qualquer mudança do código muito difícil, pois é preciso fazer modificações em vários lugares diferentes (Change Preventers)
1. **Para fazer uma pequena mudança, preciso mexer em vários métodos diferentes de uma classe** = Divergent Change
2. **Para uma mudança acontecer, preciso mexer em várias coisas em classes diferentes** = Shotgun Surgery
3. **Se eu crio uma classe filha de uma classe X, também preciso criar uma classe filha para uma classe Y** = Parallel Inheritance Hierarchies

## 4. Tem coisa meio inútil ali no meio, e a presença desse código só dificulta o entendimento (Dispensable)
1. **Código Duplicado**  = Duplicate code
2. **Classe que não ta fazendo nada** = Lazy Class
3. **Criação de código pensando no futuro** = Speculative Generality
4. **Tenho uma classe que é apenas um bocado de getter e setter e existe apenas para ser manipulada. Provavelmente ela apenas conversa com banco de dados** = Data Class
5. **Comentários** = Comments

## 5. As classes estão muito juntas, elas são meio dependentes entre si e muito acopladas (Coupling)
1. **Métodos que acessam muitos gets de outros objetos** = Feature Envy
2. **Objeto chamando objeto chamando objeto chamand...** = Message Chains
3. **Uma classe funciona apenas como intermediário para outra** = Middle man
4. **Uma classe usa métodos e campos de outra classe** = Inappropriate Intimacy
5. **Instalei uma lib que não faz tudo que eu preciso** = Incomplete Library Class

Existem diversas receitas para resolver cada um desse problema. Não conheço conteúdo em português(ainda) sobre isso, porém se seu inglês for intermediário, ou se você não tiver problema em se valer do tradutor, o site [refactoring guru](https://refactoring.guru/refactoring/smells) possui uma explicação de como resolver cada um desses problemas com diversos exemplos.

Se você quiser/conseguir investir um pouco mais, aconselho a ter [esse livro](https://amzn.to/2pXXMbn) caso você costume a programar em linguagens tipadas, como java, ou [esse](https://amzn.to/2PXx6Cm) para linguagens dinamicas tipo Ruby.
A série refactoring é bastante detalhada, com diversas receitas e exemplos de cada caso.

----

*imagem do post de [Alex Andrews](https://unsplash.com/@alex_andrews)*