---
layout: post
title:  Identificando sujeiras no código - Formatação, Objetos e estrutura de dados e erros
date:   2019-09-28 20:20:00 +0300
image:  cleancode10.jpg
tags:   Clean-Code
---
Esse post é o segundo de uma série de 3 postagens sobre **[clean code](https://www.amazon.com.br/gp/product/8576082675/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=8576082675&linkCode=as2&tag=rachc-20&linkId=0d5e2c3e67461b310960dc69f64f1d9c)**.

Aqui falaremos sobre formatação de código, objeto e estrutura de dados, e como lidar com erros.

Sei que algumas dessas coisas podem parecer bobas, mas muita coisa interessante é discutida dentro desses tópicos.

Em formatação eu falo, dentre outras coisas, sobre como **ordenar suas funções e variáveis**. Quando falo sobre objetos e estrutura de dados, falo sobre como **proteger as entranhas do seu objeto**. E sobre erros, falo um pouco sobre **preparar seu código para lidar com problemas** quando eles acontecerem.

[No primeiro post]({{ site.baseurl }}/2019/09/28/Identificando-sujeiras-no-codigo-nomes-funcoes-e-comentarios/) falei um pouco sobre **nomes, funções e comentários**. Aconselho a dar uma lida (: (Ou não! Você não é obrigado! Rebele-se!)

A intenção é finalizar a série no próximo post, falando sobre como trabalhar com código de terceiros, também como organizar seus testes unitários e, por fim, como organizar melhor suas classes.

Vamos lá!

# Formatação
![Trecho do vídeo "formation" da Beyoncé]({{ site.baseurl }}/images/cleancode11.gif)
*Formation*

Ao olhar para um código que não está bem formatado, a primeira impressão que temos, é que todo o código não está muito bom, e acabamos não o levando a sério.

Quando o assunto é formatação, não olhamos apenas a indentação, mas também como os blocos de código estão organizados.

Podemos dividir o tópico `formatação` em duas partes: Formatação vertical e formatação horizontal

## Formatação vertical
Uma matéria de jornal começa com um bom headline que informe ao leitor do que aquela matéria se trata. O primeiro parágrafo é o equivalente a uma sinopse, que você consegue ter uma idéia do tom da matéria, e a medida que você lê, vai descobrindo mais detalhes.

Um bom código deve ser assim também. O nome do módulo deve ser claro para que as pessoas saibam do que ele se trata, logo depois vem conceitos de mais alto nível juntamente com algoritmos, e a medida que o código vai se desenrolando, devem vir funções mais baixo nível com detalhes do código fonte.

O ideal é que o código seja dividido em módulos, com um espaço entre cada módulo, e eles devem seguir uma ordem, para que o código conte uma história.

Primeiro deverão ser declaradas as variáveis e depois as funções dependentes. A única coisa que deve burlar essa ordem, são funções que tenham uma afinidade conceitual.

No exemplo abaixo, consigo mostrar a ordem que as coisas deveriam ser chamadas, e quando falo em funções que possuam afinidade conceitual, falo de `is_chocolate?` e `is_not_chocolate?`

(Gente, foca na ordem que as funções aparecem e não nos nomes)

```ruby
something
somewhere = some_place

def ice_cream
  banana(something_1)
  cheese(something_2)
end

def banana(something_3)
  return "something" if is_chocolate?(something_3)
  avocado(something_3)
end

def is_chocolate?(something_4)
  #algum código
end

def is_not_chocolate?(something_4)
  #Outro código
end

def avocado(something_5)
  #Mais um cósdigo
end

def cheese(something_6)
  #mais outro código
end
```

Ainda falando sobre ordem que as coisas aparecem no código, vale lembrar que variáveis de controle de loop são declaradas dentro do próprio loop:

`for(var i = 0; i > something; i++)`

## Horizontal
É uma boa prática evitar linhas de código muito compridas. Algumas pessoas gostam de usar um limite de 80 caracteres, outras de 100 e está tudo bem, porém mais de 120 caracteres pode ser sinal de descuido.

Outra boa prática é agrupar coisas que não queremos separar, e não separar nas coisas que queremos agrupar. Parece óbvio falar assim, mas as vezes não é isso que acontece na prática:

Na expressão `def algo(argumento)` nós queremos que `algo`e `(argumento)` fiquem juntos, pois são a mesma coisa. Se colocarmos um espaço entre eles, `def algo (argumento)`, passamos uma idéia que são coisas distintas.

Já em `int algo = outro_algo` nós estamos falando que `algo`e `outro_algo` são coisas distintas, e quando escrevemos `int algo=outro_algo`, perdemos um pouco dessa distinção.

Outra coisa importante a destacar sobre formatação horizontal é como alinhamos nossas variáveis.

As vezes sentimos necessidade de alinhar:
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

1. Tiramos a atenção do que importa. No exemplo acima, acabamos não olhando para o tipo da variável. Apenas para o seu nome.
2. Geralmente se usamos ferramentas do editor para formatar o código, ele arranca esses espaços, e perdemos nossa formatação.
3. Se percebemos que precisamos alinhar as variáveis, é porque temos variáveis demais e, provavelmente, conseguimos dividir o que estamos fazendo em mais pedaços.

Assim a forma mais indicada de alinhar as variáveis é:

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
![Gif da Miss Piggy, dos muppets, forçando as barras de uma prisão para sair]({{ site.baseurl }}/images/cleancode12.gif)
*Vamos lidar melhor com esses acessos*

Se gostamos de variáveis privadas, porque sempre colocamos getters e setters? Como lidar com o acesso às nossas classes?

## Abstração de dados
Quando usamos um método ou uma classe, não precisamos saber como eles são implementados. O que nos interessa são seus retornos.

Nos exemplos abaixo conseguimos ver a diferença entre uma classe com variáveis públicas, e uma interface com várias políticas de acesso.

```java
public class Point(){
  public double x;
  public double y;
}
```
Segundo exemplo:
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
Repare que no segundo exemplo, nós não temos nenhum método do tipo `setX` ou `setY`. Se quisermos criar um ponto, precisamos passar ambos como uma dupla, usando `setCartesian`.

O primeiro exemplo nos força a lidar com cada variável separadamente, mesmo elas sendo uma dupla.

Ocultar implementação não é apenas uma questão de esconder variáveis, é uma questão de abstração. Uma classe não é apenas um emaranhado de getters e Setters. Elas manipulam dados, e essa deveria ser sua essência.

## Estrutura de dados ou Objetos?
Objetos escondem-se por trás de abstrações, e suas funções públicas servem para manipular os seus dados.

Em contrapartida, em uma estrutura de dados, tudo é exposto.

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

Qualquer nova `shape` que quisermos criar, é preciso ir na classe `Geometry` e adicionar mais um `if` na função `area`.

Nesse caso em específico, poderíamos nos valer de herança para implementar a função `area` em cada objeto, e não na classe `Geometry`.

Em Orientação a Objeto, é fácil criar novas classes sem modificar funções diferentes, mas por outro lado, em POO é difícil adicionar novas funções, pois isso impacta todas as classes filhas.

Ocasionalmente, é mais importante adicionar novas funções do que criar classes filhas. Nesses casos, não tenha medo de usar código procedural.

O importante é não misturar estrutura de dados com objetos, porque nesse caso nós teríamos o pior dos dois mundos. O resultado disso são estruturas de difícil herança, e difíceis de modificar funções existentes.

## A Lei de Demeter
A lei de Demeter fala que um módulo não deve saber as entranhas do módulo que ele manipula.

Um método não deve invocar métodos ou objetos de outra classe.

Olhando o exemplo abaixo:
```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```
se `ctxt` fosse um objeto, para acessar o `getAbsolutePath()` ele iria precisar do retorno do `getStracthDir()`, que por sua vez iria precisar do retorno de `getOptions()` para funcionar. Ou seja, as funções são dependentes uma da outra para funcionar.

Conseguiríamos resolver esse problema dessa forma:

```java
Options opts = ctxt.getOptions();
File scracthDir = opts.getScratchDir();
final String outputDir = stracthDir.getAbsolutePath();
```

Porém, devemos nos atentar que se o exemplo acima fosse uma estrutura de dados, não seria um problema, ex: `first_name = full_name.to_downcase.split(" ").first`.

O primeiro exemplo dado, temos um problema de acesso de informação, e isso viola a lei de Demeter. Já no exemplo acima, estamos transformando essa informação, e não há nenhum problema nisso.

## Transferência de dados
Quando falamos em transferência de dados, estamos nos referindo a objetos que possuem apenas variáveis, mas nenhum método.

Algumas pessoas gostam de deixar todos os atributos privados, e acessá-los através de `getters` e `setters`.

A única real vantagem disso, é deixar o código mais voltado para `Programação Orientada a Objeto`, e preparado para um possível crescimento.

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
Active record funciona como traduções diretas do banco de dados. Geralmente são `Objetos de Transferência de Dados`, com alguns métodos navegacionais como `save` ou `find`.

Acontece que, quando tentamos adicionar regras de negócio e deixá-lo mais robusto, acabamos por criar uma estrutura híbrida , meio objeto, meio estrutura de dados que, como vimos acima, é sempre ruim.

A solução para esse problema seria tratar o Active Record como uma estrutura de dados, e criar objetos separados que possuam regras de negócio e outros dado internos.

# Lidando com erros
![Gif de várias popup de erro, escrito "fail", em uma tela do windows]({{ site.baseurl }}/images/cleancode13.gif)
*Faaaail*

Lidar com erro é importante, mas quando o tratamento de erro deixa a lógica obscura e confusa, ele está sendo feito de forma errada.

Antigamente as pessoas tentavam checar por todos os tipos de erros de uma vez, e isso trazia muitos problemas além dos citados acima.

Outro problema dessa abordagem é que você precisa pensar em todos os cenários antes de escrever o código, que não só prejudica a legibilidade, como é fácil de esquecer de tratar o erro depois.

Por isso a forma mais interessante de lidar com erro é jogar uma exception mais genérica.

Quando jogamos exceptions, precisamos ter o cuidado de deixar as mensagens claras para que você, desenvolvedor, saiba como identificar o problema depois que ele ocorre.

Uma dica para escrever códigos que lidem bem com erro, é escrever seu `try-catch` primeiro.

Pensa que tudo que está no `try catch` deve ser independente e desacoplado do restante da aplicação pois, se ele aquele pedaço de código der problema, provavelmente não queremos quebrar todo o resto.

Escrevendo o `try-catch` primeiro, ajuda a deixá-lo independente.

## Uma prévia sobre lidar com código alheio

Quando usamos um código de terceiros, o ideal não tratar os erros dele direto no seu código.

O ideal é encapsular essa lib em uma classe só pra ela, e tratar os erros lá dentro.

## Null/Nil
Quando trabalhamos com `null` no nosso código, criamos um problema difícil de debugar.

Quando alguma coisa dá errada, ele solta uma exceção genérica, nos informando que algo de errado não está certo. Essa exceção é tão genérica, que encontrar sua raiz é bem difícil.

As vezes precisamos verificar se algo não existe e ficamos tentado a verificar se `algo == null`.Nesse caso, procure saber se não existem outras formas de verificação. Provavelmente existem.

Se for uma lista, podemos verificar se a lista está vazia, se for uma String, conseguimos verificar se ela está em branco e por ai vai.

E, o mais importante, não passe null como argumento. Quando fazemos isso, as chances de dar um `null pointer exception` (e equivalentes) são muito maiores.
----
*imagem do post de [Jazmin Quaynor](https://unsplash.com/@jazminantoinette)*