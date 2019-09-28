---
layout: post
title:  Identificando sujeiras no código - Nomes, Funções e comentários
date:   2019-09-28 10:30:00 +0300
image:  cleancode00.jpg
tags:   Clean-Code
---
O problema de termos um código não muito limpo, é a medida que vamos dando manutenção, ele vai ficando **mais e mais complexo**, e sua compreensão fica comprometida.
 
Quando eu falo em código complexo, não me refiro a regras de negócio complexas, e sim complexidade de **entendimento e legibilidade**. Em gastar um tempo considerável tentando entender o que era `p`, ou o que a função `read(p, i)` fazia por você não ter contexto. (o que ela lia? o que são esses parâmetros? aaahhh /o\ )
 
Quando chegamos em um ponto que gastamos muito mais tempo tentando entender um código antigo, do que pensando na implementação de um código novo, pode ser um sinal que alguma coisa poderia melhorar. **Gastar mais tempo do que gostaríamos em uma tarefa nos deixa frustrados**.
 
Quando a manutenção de um código é difícil, nós paramos de se importar com a sua qualidade e, consequentemente o sujamos mais, deixando o código ainda mais complexo, e quanto mais complexo e difícil ele fica, e nossa produtividade cai para algo próximo de 0. Corremos sempre o risco de consertar um problema, criar mais 3 no processo. Acaba que **ficamos com medo de tocar em código que funciona "magicamente"** (porque obviamente não fazemos idéia do que ta acontecendo).
 
As consequências disso são códigos tão difíceis, de manutenção tão impossível que as **empresas acabam fechando** (eu já vi isso acontecer. O único produto da empresa possuía vários bugs, era impossível consertar um bug sem criar mais alguns, e a insatisfação dos clientes chegou em um nível que eles não queriam mais usar o produto).
 
## Se sabemos que isso é tão prejudicial, por que fazemos isso?
**Pressa.**
 
Geralmente temos prazos apertados, e entregamos de qualquer jeito para podermos passar pra próxima tarefa.
 
## Quero evitar que isso aconteça. Comofas ?
A idéia dessa série de posts é resumir os principais tópicos do [livro clean code](https://www.amazon.com.br/gp/product/8576082675/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=8576082675&linkCode=as2&tag=rachc-20&linkId=0d5e2c3e67461b310960dc69f64f1d9c). Querendo mais detalhes, aconselho bastante a dar uma lida (:
 
Essa é a **primeira parte** da série sobre clean code. A idéia, nesse primeiro momento é falar sobre nomes, funções e comentários.
 
Ainda pela frente, vamos falar também sobre formatação, Objetos e estrutura de dados, Como lidar com erro, sobre limites (como lidar com código alheio), testes unitários, e como organizar bem sua classe.
 
Sei que vai ficar faltando uma parte importante sobre code smells, mas esse tópico é muito amplo, e dá material para uma outra série
 
Vamos começar?
 
# Batizem bem o seu código!
 
![gif escrito "hello, my name is"]({{ site.baseurl }}/images/cleancode01.gif)
*Por favor, me dê um bom nome*
 
## Use nomes que demonstrem o seu código faz:
`int d` pode ser `int days`
 
`while x == 7`, poderíamos usar `while time == DAYS_OF_WEEK`
 
## Evite desinformações.
se `accountsList` não for uma lista, ela pode ter um nome melhor.
 
Variáveis com diferenças muito sutis de nome também entram nesse tópico como
`VeryNiceControllerToDealWithStrings` e `VeryNiceControllerToStoreStrings`.
 
Quanto tempo você gasta pra conseguir perceber a diferença entre as duas variáveis?
 
## Faça boas distinções das suas variáveis.
se temos duas variáveis, uma chamada `banana` e outra `theBanana`, como vamos saber quem é quem? a mesma coisa vale para `CustomerData`, `Costumer` e `CostumerInfo`. Qual a diferença entre elas?
 
## Use nomes pronunciáveis
Facilita a comunicação interna do time. Se você fizer um pair, não precisa ficar se referindo a uma variável como `wdpw`. Um `workDaysPerWeek` soa melhor (:
 
## Use nomes que você consegue buscar facilmente.
já tentou usar ctrl+f em uma variável que você não consegue lembrar o nome, ou que era tão comum, que você só não conseguiu? já tentou procurar por uma variável chamada `f`?
 
## Evite prefixos!
Também fica difícil lidar com busca e, principalmente, fazer nosso autocomplete funcionar como queremos se várias variáveis começam com `ic`
 
## Evitem mapa mental
quando usamos uma quantidade gigantesca de variáveis que não fazem muito sentido, precisamos decorar mentalmente o que elas significam e acabamos por ter dois trabalhos: saber o que cada variável é, e entender o que o código está fazendo.

```ruby
user.each do |u|
  u.each do |s|
    s.each do |h|
      h.name
    end
  end
end
```
(fora que esse bloco de código também tem outro problema de each dentro de each dentro de each, mas falaremos depois)

## Nomes de classe
Devem ser substantivos e não verbos.

## Nomes de métodos
Devem ser verbos, e não substantivos.

## Adicione contexto!
Se você esbarrar em uma variável `state` sozinha, provavelmente vai ter dificuldades de saber o que ela significa.

Mas se essa variável está junto de `street` e `number` fica mais fácil entender o que ela faz.

## Adicione contexto, mas não gratuitamente.
Supondo que estamos trabalhando em um projeto chamado `chocolate ice cream`. Não precisamos adicionar `CIC` na frente de todas as variáveis.

Ao mesmo tempo, que dentro de uma classe `User` não precisamos chamar as variáveis como `userAdress` ou `userName`.

# Funções
![Gif do muppet cozinheiro, batucando panelas, com a cozinha toda em movimento]({{ site.baseurl }}/images/cleancode02.gif)
*Function, Function, Function*

## Que sejam pequenas
Elas devem ser pequenas e legíveis, com poucos nesteds. O ideal é que tenham 1 ou 2 níveis de indentação.

## Fazem apenas uma coisa
Elas devem fazer apenas uma coisa e é isso.

Mas como saber se elas fazem apenas uma coisa?

```ruby
def verifyNameAndLogout
  if rightName?
    showMessage
  end

  logout
end
```
Olhando o código, parece que ela faz três coisas.
1. Verifica se o nome é correto
2. Caso positivo, exibe uma mensagem
3. Independente do nome ser correto ou não, ele faz o logout.

Entendemos que ela faz o que se propõe fazer com apenas um nivel dentro da função. Então podemos dizer que ela faz apenas uma coisa.

A única alteração que poderíamos fazer nesse código seria levar o if para outra função, mas nesse caso seria apenas uma relocação de if, sem nenhuma vantagem adicional. 

Funções não podem ser divididas em seções. Caso você consiga fazer essa divisão, é sinal que ela pode ser mais de uma função.

## Um nível de abstração por função

Quando falamos de níveis de abstrações, podemos classificar o código em três níveis:
1. alto: `getAdress`
2. médio: `inactiveUsers = Users.findInactives`
3. baixo: `.split(" ")`

O ideal é não misturar os níveis de abstrações em uma única função. Na hora que misturamos, a função tende a ficar confusa, e não conseguimos ler o código passo a passo, de cima pra baixo, como um texto.

O ideal é que você consiga ler um código de cima pra baixo, como uma narrativa.

ex:

1. Para incluir o setup e teardown nós incluímos setups, dai incluímos o conteúdo da página de teste e então os teardown.
2. Para incluir o setup, nós incluimos a suite de setup, e incluimos o setup padrão.
3. Para incluir o setup de testes, nós procuramos pela hierarquia... 

(e por aí vai)

## Evitar Switch Statements e cadeias de if/else
O motivo que devemos evitar, é que além de serem muito grandes e fazerem muitas coisas, eles violam o princípio "open-closed" de orientação objeto (ou o famoso `O` de S.O.L.I.D ). Uma entidade deveria ficar fechado para alterações e aberto para ampliação. (leia-se entidade como classes, módulos, funções e etc)

Quando se tem vários if/else/switch, qualquer caso a mais que queremos contemplar vai precisar ser adicionado no código que queremos, o que torna o código frágil e difícil de testar.

As vezes precisamos verificar várias situações diferentes e tomar ações diferentes dependendo da situação.

Uma forma de resolver isso é com polimorfismo. 

Dando um exemplo bem simples

```java
foreach (var animal in zoo) {
  switch (typeof(animal)) {
    case "dog":
      echo animal.bark();
      break;

    case "cat":
      echo animal.meow();
      break;
  }
}
```
pode virar
```java
foreach (var animal in zoo) {
  echo animal.speak();
}
```
No exemplo acima, por debaixo dos panos, criamos uma classe de animal que tem o método speak. Criamos um objeto cachorro e gato, damos o comportamento speak para cada um, e é isso.

As vezes polimorfismo pode ser um pouco demais para resolver esse problema. As vezes conseguimos apenas dar uma mexida na lógica, criamos umas funções a mais resolvemos nosso problema.

## Diminua a quantidade dos seus argumentos
Quanto menos argumentos, melhor.

Quando temos muitos argumentos, a complexidade da nossa função e, consequentemente dos seus testes, aumenta bastante.

Fora que comumente trocamos a ordem dos argumentos quando vamos usar a função.

Ocasionalmente podemos diminuir os argumentos criando uma classe.

As funções com vários argumentos podem ser transformados em métodos dessas classes, e os argumentos muitas vezes podem ser propriedades delas, e assim não precisamos passar argumento algum.

Em alguns casos não precisamos de tanto, então podemos usar outras formas de diminuir o número de argumentos, como agrupá-los.

`Circle makeCircle(double x, double y, double radius)` pode ser transformado em `Circle makeCircle(Point center, double radius)`

## Funções são verbos, argumentos substantivos
Quando nomeamos funções como verbos, e deixamos argumentos como substantivos, fica mais legível e fácil de entender o comportamento que esperamos, como no exemplo `write(name)`

A ordem dos argumentos também é importante para você não esquecer na hora de usar a função.

imagina acertar o uso dessa função se os argumentos viessem trocados: `assertExpectedEqualsActual(expected, actual)` 

## Evite flags como argumento.
Evite passar uma booleana como argumento, pois torna a refatoração difícil.

Tente fazer a verificação quando for chamar a função, e não dentro dela.

## Uma função não deve possuir efeitos colaterais.
Se temos uma função chamada `checkPassword`, ela não deveria fazer um `login` dentro dela. 

Em nenhum momento nós fomos preparados para esse login. Isso pode resultar em efeitos colaterais indesejados (fora a dificuldade de testar).

## Argumentos de output
De forma geral funções de output deveriam ser evitadas. Se a função precisa mudar o estado de algo, que mude o próprio objeto.

As vezes esbarramos em funções do tipo `appendFooter(s)` e ficamos na duvida se o argumento é um imput (`s` é o footer e nós vamos anexar ele em algum lugar?) ou output (vamos anexar `s` ao footer?).

Se for o primeiro caso, em que s é o footer, o melhor é fazer `objectInstance.appendFooter`

## Separação de queries de comando.
Uma função deveria mudar o estado de um objeto, ou retornar uma informação sobe o mesmo.

```ruby
if name?
  name.change
else
  false
end
```

Uma função que faz uma modificação e retorna true ou false se a modificação ocorrer, deve ser trocado por duas. Uma que verifica, e outra que troca.

## DRY - Don't repeat yourself
O problema de código repetido é que, além de inflar o código, qualquer alteração que precisemos fazer em um ponto, vai ser necessária atualização dos demais pontos do código.

## Evite retornar códigos de erro
Quando métodos retornam mensagens de erro, ele está violando sutilmente a separação de Queries de comando. Quando colocamos um erro dentro de um bloco de `if` pode causar alguma confusão, já que ele ou ele faz algo, ou retorna um erro.

# Comentários
![Cachorrinho sendo entrevistado. Legenda fala "no coments"]({{ site.baseurl }}/images/cleancode03.gif)
*No coments*
**Evite**

Comentários em código geralmente trazem mais mal do que bem. Muitas vezes um comentário sobre o que uma variável faz, ou detalhes de como funcionam as coisas podem:
1. Ficar defasadas causando confusão para você do futuro (ontem mesmo perdi precioso tempo por conta de um comentário defasado)
2. Podem ser substituídos por um nome melhor.
3. Poluem desnecessariamente o código.

Claro que evitar != de proibir. Se você é um programador java e está fazendo algo publico, javadocs é importante. As vezes é interessante você explicar algumas coisas (quanto tempo você leva pra entender uma regex quando esbarra em uma?), mas 99% das vezes, comentários podem ser evitados.

# Mas como eu vou escrever código se tenho que prestar atenção em todas essas regras?
![Pikachu surpreso]({{ site.baseurl }}/images/cleancode04.gif)
*No coments*

Assim como um bom artigo, que você coloca as idéias em um papel e depois vai refinando o texto, com código acontece o mesmo.

Primeiro faz funcionar, depois refatora.
Uncle bob, em clean code, defende que a melhor ordem de escrever um código é:
1. Testes unitários.
2. Código que funcione.
3. Refatoração para um código limpo.

Escreve todos os testes, faz funcionar e, tendo a certeza que tudo funciona, refatora removendo as sujeiras e aplicando design patterns.

Particularmente eu não tenho o hábito de fazer testes antes do código (TDD), porém, testar antes de refatorar é imprescindível para ter a certeza que seu código sempre funcione, mesmo depois de tanta modificação.

O livro (Clean code)[(https://www.amazon.com.br/gp/product/8576082675/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=8576082675&linkCode=as2&tag=rachc-20&linkId=0d5e2c3e67461b310960dc69f64f1d9c)] é muito focado para as más práticas de código. Nele são detalhados os diversos problemas, e é falado por cima nas formas de se corrigir. 

Quem quiser se aprofundar mais em como corrigir os problemas, pode ler o livro (refactoring)[https://www.amazon.com.br/s/ref=as_li_ss_tl?k=refactoring&__mk_pt_BR=%C3%85M%C3%85%C5%BD%C3%95%C3%91&ref=nb_sb_noss_2&linkCode=ll2&tag=rachc-20&linkId=65987f7573c86a94cd2bd08051b1e328&language=pt_BR], que são detalhadas e exibidas diversas formas diferentes de eliminar todas as sujeiras do código.

----
*imagem do porst de [Sarah Dorweiler](https://unsplash.com/@sarahdorweiler)*