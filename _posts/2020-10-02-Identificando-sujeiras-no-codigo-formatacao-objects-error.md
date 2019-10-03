---
layout: post
title:  Identificando sujeiras no código - Formatação, Objetos e estrutura de dados e erros
date:   2019-09-28 20:20:00 +0300
image:  cleancode10.jpg
tags:   Clean-Code
---
Esse post é o segundo de uma série de 3 sobre clean code.

Hoje falaremos sobre formatação de código, objeto e estrutura de dados e como lidar com erros.

Sei que algumas dessas coisas podem parecer bobas, mas em formatação eu falo, entre outras coisas, sobre como ordenar suas funções em um arquivo. Quando falo sobre objetos e estrutura de dados, falo sobre como proteger as entrenhas do seu objeto. E sobre erros, falo um pouco sobre proteger seu código para que ele esteja preparado para lidar com erros quando eles acontecerem.

[No primeiro post]({{ site.baseurl }}/2019/09/28/Identificando-sujeiras-no-codigo-nomes-funcoes-e-comentarios/) falei um pouco sobre nomes, funções e comentários. Aconselho a dar uma lida (: (Ou não! Você não é obrigado! Rebele-se!)

A idéia é terminar a série no próximo post, em que falarei sobre como trabalhar com código de terceiros, também como organizar seus teses unitários e, por fim, como organizar melhor suas classes.

Vamos lá!

# Formatação
Quando olhamos para um código que não esta bem formatado, a primeira impressão que temos, é que o código não está muito bom, e acabamos levando menos a sério do que deveriamos. Nós olhamos não apenas a identação, mas também como os blocos de código estão organizados.

Quando falamos em formatação de código, conseguimos dividir o assunto em duas partes: Formatação vertical e formatação Horizontal

## Vertical
Uma matéria de jornal começa com um bom headline, que informe ao leitor do que aquela matéria se trata. O primeiro parágrafo é o equivalente a uma sinopse, que você consegue ter uma idéia do tom da matéria, e a medida que você lê, vai descobrindo mais detalhes.

Um bom código deve ser assim também. O nome do módulo deve ser claro para que as pessoas saibam do que ele se trata, logo depois vem os conceitos mais high-level e algoritmos, e a medida que vai passando o código, deve vir funções mais low level e detalhes do código fonte.

O ideal é que o código seja dividido em módulos, com um espaço entre cada módulo, e eles devem seguir uma ordem, para que o código conte uma história.

Primeiro deverão ser declaradas as variáveis e depois as funções dependentes. é aconselhável que funções que tenham uma afinidade conceitual fiquem agrupadas:

(Gente, foca na ordem!)

```ruby
something
somewhere = some_place

def ice_cream
  banana(something_1)
  cheese(something_2)
end

def banana(something_1)
  is_chocolate?(something_1)
  avocado(something_1)
end

def is_chocolate?(something_1)
  something_1
end

def is_not_chocolate?(something_1)
  something_10
end

def avocado(something_1)
  self.something = something_1
end

def cheese(something_2)
  self.somewhere = something_2
end
```

Sobre ordem que as variáveis são incluidas no código, é importante lembrar que as variáveis de controle de loop são declaradas dentro de um loop.

`for(var i = 0; i > something; i++)`

## Horizontal
É interessante evitar linhas muito compdidas de código. Algumas pessoas gostam de usar um limite de 80 caracteres, outras de 100 e está tudo bem mas, mais de 120 caracteres pode ser sinal de descuido.

É interessante agrupar coisas que não queremos separar, e não usar espaço nas coisas que queremos agrupar. Parece óbvio falar assim, mas as vezes isso não acontece na prática.

Na expressão `def algo(argumento)` nós queremos que `algo`e `(argumento)` fiquem juntos, pois são a mesma coisa. `def algo (argumento)` passa uma idéia que são coisas distintas

Já em `int algo = outro_algo` nós estamos falando que `algo`e `outro_algo` são coisas distintas, e quando fazemos `int algo=outro_algo`, perdemos um pouco dessa distinção.

Outra coisa importante a destacar sobre alinhamento horizontal é como alinhamos as coisas.

As vezes sentimos necessidade de alinhar as coisas
```java
private Socket        socket;
private InputStream   input;
private OutputStream  output;
private Request       request;
private Response      response;
private boolean       hasError;
private long          requestParsingTimeLimit;
private long          requestedProgress;
```

Porém temos três problemas quando fazemos isso.

1. tiramos a atenção do que importa. No exemplo acima, acabamos não olhando para o tipo da variável. Apenas para o seu nome
2. Geralmente se usamos ferramentas do editor para formatar o código, ele arranca esses espaços, e perdemos tudo.
3. Bonus e o mais importante: Se a gente percebe que precisamos alinhar as variáveis, é porque temos variáveis demais e, provavelmente, temos que dividir o que estamos fazendo em mais pedaços.

Assim é a forma mais indicada de alinhar as variáveis

```java
private Socket socket;
private InputStream input;
private OutputStream output;
private Request request;
private Response Response;
private boolean hasError;
private long requestParsingTimeLimit;
private long requestedProgress;
```

# Objetos e estrutura de dados
Gostamos de variáveis privadas, porque não queremos ninguém dependendo dessas variáveis. Então porque sempre colocamos getters e setters se as variáveis são privadas?

## Abstração de dados
é desnecessário saber como os métodos são implementados.

Nos exemplos abaixo vimos a diferença entre uma classe com variáveis públicas, e uma interface com várias políticas de acesso

```java
public class Point(){
  public double x;
  public double y;
}
```
```java
public interface Point(){
  double getX();
  double getY();
  void setCartesian (double x, double y);
  double getR();
  double getTheta();
  void setPolar(double r, double theta);
}
```
Repare que no segundo exemplo, nós não temos nenhum método do tipo `setX` ou `setY`. Se eu quisermos criar um ponto, preciso passar ambos como uma dupla, usando `setCartesian`.

O primeiro nos força a lidar com cada variável separadamente, mesmo elas sendo uma dupla

Ocultar implementação não é apenas uma questão de esconder variáveis, é uma questão de abstração. Uma classe não é apenas um emaranhado de getters e Setters. Elas manipulam dados, e essa deveria ser sua escência.

## Estrutura de dados ou Objetos?
Objetos se escondem por trás de abstrações e suas funções públicas servem para manipular os seus dados.

Enquanto em uma estrutura de dados, tudo é exposto.

```ruby
class Square < Shape
  Point top_and_left
  side
end

class Circle < Shape
  Point center
  radius;
end

class Geometry
  PI = 3.141592

  def area(geometric_form)
    if geometric_form.is_a?(Square)
      # calcule a área de um quadrado
    elsif geometric_form.is_a?(Circle)
      # Calcule a área de um circulo usando PI
    end
  end
end
```

No exemplo acima nós temos uma estrutura de dados com o seguinte problema:

Em que qualquer nova forma que quisermos criar, precisamos ir na classe "Geometria" e adicionar mais um `if` na definição da área.

Nesse caso em específico, poderiamos nos valer de herança para implementar a função área em cada objeto, e não na classe `Geometry`

Em Orientação a Objeto, é fácil criar novas classes sem modificar funções diferentes, mas por outro lado, em POO é difícil adicionar novas funções, pois isso impacta todas as classes filhas.

Ocasionalmente, é mais importante adicionar novas funções do que criar classes filhas. Nesses casos, não tenha medo de usar código procedural.

O importante é não misturar estrutura de dados com objetos, porque nesse caso nós teriamos o pior dos dois mundos. O resultado disso são estruturas dificeis de criar novas classes, e difíceis de modificar funções existentes.

## A Lei de Demeter
A lei de Demeter fala que um módulo não deve saber as entranhas do módulo que ele manipula. 

Um método não deve invocar métodos ou objetos de outra classe.

Olhando o exemplo abaixo:
```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```
se `ctxt` fosse um objeto, pra acessar o `getAbsolutePath()`, eu teria que passar por vários métodos desse objeto, pra conseguir um retorno. Então, para o `getAbsolutePath() `funcionar, ele iria precisar do retorno do `getStracthDir()`, que por sua vez iria precisar do retorno de `getOptions()` para funcionar.

Como nós podemos resolver esse problema?

```java
Options opts = ctxt.getOptions();
File scracthDir = opts.getScratchDir();
final String outputDir = stracthDir.getAbsolutePath();
```

Porém, devemos nos atentar que se o exemplo acima fosse uma estrutura de dados, não teria problemas, ex: `first_name = full_name.to_downcase.split(" ").first`

As vezes esbarramos com esse tipo de código:
```java
String outfile = outputDir + "/" + className.replace(".", "/") + ".class";
FileOutputStream fout = new FileOutputStream(outFile);
BufferedOutputStream bos = new BufferedOutputStream(fout);
```
Esse código faz muita coisa e cada linha de código está dependente do anterior, ciolando a lei de Demeter.

Mas como podemos resolver?
```java
BufferedOutoutStream bos = ctxt.createScratchFileStream(classFileName)
```
Podemos esconder a estrutura de um arquivo em uma classe. Assim, ele não estaria violando a lei de Demeter.

## Transferência de dados
Quando falamos em transferência de dados, estamos nos referindo a objetos que possuem apenas variáveis com nenhum método.

Algumas pessoas gostam de deixar todos os atributos privados, e acessá-los através de `getters` e `setters`.

A única real vantagem disso, é deixar o código mais voltado para Programação orientada a objeto, e preparado para um possível crescimento.

```java
public class Person{
  private String name;
  private int age;

  public Person (String name, int age){
    this.name = name;
    this.age = age;
  }

  public getName(){
    return name;
  }

  public getAge(){
    return age;
  }
}
```

## Active Record
Active record funciona como traduções diretas do banco de dados. Geralmente são Objetos de Transferência de dados, com alguns métodos navigacionais como `save` ou `find`.

O problema acontece quadno tentamos modificar ele para adicionar regras de negócio, criando uma estrutura Hibrida, meio objeto, meio estrutura de dados

A solução seria tratar o Active Record como uma estruturade dados, e criar objetos separados que possuam regras de negócio e outros dado internos.

# Lidando com erros
Antigamente as pessoas tentavam checar por todos os tipos de erro, e, dando ruim, eles enviavam um erro para ser tratado depois.

O problema é que você precisa pensar em tudo isso antes de escrever o código, o que além de prejudicar a legibilidade, é fácil de esquecer. 

Por isso, é interessante jogar uma exception mais genérica.

O mais importante é que as mensagens de erro sejam claras o suficiente para você, desenvolvedor, saber o que está acontecendo depois.

Outra coisa interessante, é escrever seu `try-catch` primeiro. 
Pensa que tudo que está no try catch deve ser independente e desacoplado do restante da aplicação pois, se ele aquele pedaço de código der problema, provavelmente não queremos quebrar todo o restante da aplicação.

Escrevendo o `try-catch` primeiro, ajuda a deixá-lo independente.

## Uma pévia sobre lidar com código alheio

Quando usamos um código de terceiros, o ideal não tratar os erros dele direto no seu código.

O ideal é encapsular essa lib em uma classe só pra ela, e tratar os erros lá dentro

## Definindo o fluxo normal.
As vezes usamos o try catch para lidar com uma exceção de regra e negócio.

No exemplo abaixo, o retorno do valor default não é feito dentro de uma trativa de erro, e sim dentro da classe, que é onde ele deveria estar

antes:
```java
try{
  MealExpenses expenses = expenseReportDAO.getMeals(employee.getID())
  m_total += expenses.getTotal();
} catch (MealExpensesNotFound e){
  m_total += getMealPerDiem();
}
```
Depois
```java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID())
m_total += expenses.getTotal();

----
public class perDiemMealExpenses implements MealExpenses {
  public int getTotal(){
    //return the per diem default
  }
}
```

## Null/Nil
Qundo trabalhamos com null no nosso código, criamos um problema difícil de debugar. 

Quando alguam coisa da errada, ele solta uma exceção genérica, nos informando que algo de errado não esta certo, e é sempre difícil encontrar a causa raiz no nosso código.

As vezes precisamos verificar se algo não existe e ficamos tentado a verificar se `algo == null`.Nesse caso, verifique se existem outras formas de verificação. Dificilmente você não vai conseguir fazer essa troca.

Se for uma lista, podemos verificar se a lista está vazia, se for uma String, conseguimos verificar se ela está em branco e por ai vai.

E, o mais importante, não passe null como argumento. Quando fazemos isso, as chances de dar um `null pointer exception` (e equivalentes) são muito maiores.
----
*imagem do post de [Sarah Dorweiler](https://unsplash.com/@sarahdorweiler)*