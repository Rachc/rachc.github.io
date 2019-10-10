---
layout: post
title:  Identificando sujeiras no código - Código de terceiros, testes e classe
date:   2019-10-10 08:40:00 -0300
image:  cleancode20.jpg
tags:   Clean-Code
---
Quando comecei com a série sobre [clean code](https://www.amazon.com.br/gp/product/8576082675/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=8576082675&linkCode=as2&tag=rachc-20&linkId=0d5e2c3e67461b310960dc69f64f1d9c), a intenção era não apenas fixar meu conhecimento sobre o assunto, como passar pra frente o conhecimento que adquiri. 

Essa série também serve como guia rápido do livro. Quando preciso relembrar um ou outro tópico, consigo encontrar mais rápido o que procuro.

[No primeiro post]({{ site.baseurl }}/2019/09/28/Identificando-sujeiras-no-codigo-nomes-funcoes-e-comentarios/) consegui falar sobre Nomes, Funções e Comentários.


[No segundo post]({{ site.baseurl }}/2019/10/07/Identificando-sujeiras-no-codigo-formatacao-objects-error/) falei sobre formatação, Estrutura de dados e como lidar com erros.

Nesse último, falo sobre como lidar com código de terceiros, como organizar seus testes e suas classes.

Além desses 9 tópicos, o [livro clean code](https://www.amazon.com.br/gp/product/8576082675/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=8576082675&linkCode=as2&tag=rachc-20&linkId=0d5e2c3e67461b310960dc69f64f1d9c) fala sobre lidar com sistemas, sobre alguns tópicos mais voltados para a linguagem Java e ele aborda também como identificar alguns code smells.

Decidi não abordar esses tópicos pois gostaria de deixar a série mais genérica e voltada para todas as linguagens. Também por não ter tanto conhecimento prático sobre construções de sistemas do zero, e acredito que uma série exclusiva sobre refactoring poderia abordar melhor os code smells, já que no [livro clean code](https://www.amazon.com.br/gp/product/8576082675/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=8576082675&linkCode=as2&tag=rachc-20&linkId=0d5e2c3e67461b310960dc69f64f1d9c) o tema é um pouco superficial.

Dado os avisos, podemos começar com os tópicos do posts de hoje:

# Lidando com código de terceiros
Dificilmente fazemos um código 100% sozinhos. Geralmente contamos com alguma biblioteca para nos ajudar, e algumas vezes colegas contribuem com o código.

É importante aprender a lidar com os limites do código, identificando bem onde o nosso código acaba e o código do outro termina, para conseguirmos manter o sistema com maior facilidade.

Um dos problemas de se usar códigos de terceiros, é que geralmente eles possuem mais coisas que precisamos, e isso pode deixar nosso código com um comportamento estranho.

## Encapsule sua biblioteca!

A melhor forma de resolver isso, é encapsular a biblioteca que iremos usar em uma classe e puxar apenas os métodos que vamos precisar.

Outra vantagem de encapsular uma biblioteca de terceiros em uma classe própria é que ao fazer isso, conseguimos ter mais facilidade em alterar versões ou até trocar a própria biblioteca.

Vamos supor que temos uma biblioteca no nosso código que conecta a um serviço de pagamento. No dia que quisermos trocar o serviço, é possível fazer essa troca apenas na classe que criamos. Conseguimos adaptar nossos métodos sem precisar vasculhar o código inteiro para trocar o que for da biblioteca antiga para a biblioteca nova.

## Explorando códigos de terceiros.
Quando queremos adicionar uma nova biblioteca no nosso código, precisamos entender o que ela faz e como ela funciona. Colocar código que não entendemos em produção pode ocasionar em alguns problemas.

A melhor forma de descobrir o comportamento de uma biblioteca que queremos adicionar é escrever testes para situações que queremos que a biblioteca resolva e fazê-los passar. Dessa forma não só conseguimos descobrir como o código funciona, mas esses testes podem ser úteis para uma eventual troca de versão ou de biblioteca.

Se você trocar trocar a versão e rodar os testes, vai descobrir quais comportamentos não funcionam mais e corrigi-los.

Se eu puder resumir o uso de bibliotecas de terceiros no seu código em alguns tópicos, eles seriam:

* Manter uma separação clara do código de terceiros e os seus.
* Escrever bons testes para saber se o código de terceiro vai se portar da forma que esperamos.
* Evitar ao máximo que nosso código tenha contato com códigos terceiros. É melhor depender de algo que você controla do que algo que você não controla.
* Encapsular e criar adaptadores para que nosso código se refira a ele o mínimo possível, e para que nós possamos trocar a biblioteca no caso de algum problema.

# Organizando seus testes
A maior preocupação que temos ao escrever um teste deve ser a sua legibilidade.

Em um código de teste, a legibilidade pode ser até mais importante que um código em produção. 

Quando escrevemos um teste devemos evitar escrever detalhes que não fazem parte dele, para que a pessoa que precise (re)consertar esse teste no futuro saiba o que está acontecendo sem precisar entender código desnecessário.

## Use um assert por teste
O ideal é que você possua apenas um assert por teste, para que não haja dúvidas qual a parte do teste quebrou quando isso acontecer.

Essa regra pode resultar em muitos testes repetidos. Para resolver isso, você pode usar `before` com o bloco de código que se repete.

Se há muita repetição, podemos até usar mais de um assert por teste, mas com muito cuidado para manter o número de asserts pequenos.

Ou, dando uma opinião minha e não do Uncle bob, você pode não resolver esse problema, já que muito [DRY](https://pt.wikipedia.org/wiki/Don%27t_repeat_yourself) pode prejudicar a legibilidade do código, e a legibilidade sempre deve ser sua prioridade quando falamos de teste.

Talvez, uma regra melhor seja que cada teste deve testar apenas um conceito.

É muito importante que ele não teste mais de uma coisa por vez para o teste se manter conciso, e você consiga manter o controle do que está acontecendo.

O Uncle bob usa um acrônimo para ajudar a lembrar regras de um teste limpo.

## F.I.R.S.T
* **F**ast (Rápido): Testes devem ser rápidos, pois se eles forem longos não vamos querer rodar os testes.
* **I**ndependent (Independente): Um teste nunca deve depender de outro teste para rodar. Deve ser possível rodar todos os testes aleatoriamente e sem que eles quebrem.
* **R**epeatable (Repetível): Um teste precisa rodar em qualquer ambiente, seja em QA, em Produção ou no computador da sua casa. Ele deve ser repetível.
* **S**elf-validating (auto validados): O resultado de um teste deve ser sempre um boolean, que informe se ele passou ou não.
* **T**imely (oportuno): Eles tem um momento certo para serem escritos, que é antes de teste de produção. Se você deixar para depois, pode ser que você ache o teste muito difícil de ser testado e não testar (TDD)

# Organizando suas classes
Em java, o padrão é que uma classe deve seguir a seguinte ordem:

1. Lista de variáveis
2. Constantes estáticas públicas
3. Variáveis estáticas privadas
4. Variáveis privadas de instância.
5. Funções públicas
6. Funções privadas chamadas pelas funções públicas devem ficar imediatamente abaixo da função pública que a chama

Já em Ruby e em outras linguagens, todas as classes privadas devem estar agrupadas, e escritas depois de todo o código público. O ideal é que elas sigam a ordem que foram chamadas pelos métodos públicos.

## Classes devem ser pequenas.

Quando falamos que uma **função** deve ser pequena, falamos que ela deve ter poucas linhas.

Quando falamos que uma **classe** deve ser pequena, queremos dizer que ela precisa seguir o princípio de responsabilidade única.

## Mas como saber se uma classe possui responsabilidades demais?

O primeiro sinal que uma classe possui responsabilidades demais é o nome dela.

Classes devem ser nomeadas de uma forma explicativa, que você consiga entender o que ela faz. Se você precisa por um "and", um "or", um "if" e um "but", ela já está fazendo mais de uma coisa.

no exemplo abaixo podemos ver que a classe `SuperDashboard` procura pelo ultimo componente focado **e** altera a versão da build, dessa forma tendo duas responsabilidades.

```java
public class SuperDashboard {
  public Component getLastFocusedComponent();
  public int getMajorVersionNumber();
  public int getMinorVersionNumber();
  public int getBuildNumber();
}
```

Uma forma de resolver esse problema é criar uma classe para cada responsabilidade.

Algumas pessoas reclamam que quando temos muitas classes, fica difícil encontrar o que queremos.

Mas pensa que se temos uma classe com muitas coisas, também vamos ter dificuldade de encontrar o que procuramos. No caso em que temos muitas classes, é como se tivéssemos várias gavetas etiquetadas com seus conteúdos. Já no segundo temos poucas gavetas entulhadas de coisa.

## Um pouco sobre coesão
Em um mundo ideal, as funções dentro de uma classe deveriam falar apenas com as variáveis criadas naquela classe mas, como isso é impossível, precisamos que nossas classes sejam coesas.

## Novas classes como resultado de refatoração.

Podemos observar a existência de um fluxo quando estamos refatorando um código, que pode resultar na criação de mais classes:

1. Quando quebramos uma função grande em várias menores, sentimos a necessidade de compartilhar variáveis.
2. Queremos criar variáveis de instância de classe para que as funções não precisem de parâmetros, porém qual o motivo de criarmos variáveis de instância que a minoria das funções usarão? Isso não parece muito interessante.
3. Será que isso não é um indicativo que novas classes poderiam ser criadas? :D

## Organizando a classe para mudanças.
Precisamos lembrar que uma classe deve seguir o princípio "open-closed", sendo aberto a ampliação e fechado para mudanças.

As vezes nossa classe engloba uma biblioteca de terceiros que está sob constante mudança. Quando isso acontece, podemos resolver isso com uma classe pai para conectar com a biblioteca, e criar várias classes filhas com as novas implementações da biblioteca a medida que elas forem acontecendo.

----
*imagem do post de [Sarah Dorweiler](https://unsplash.com/@sarahdorweiler)*