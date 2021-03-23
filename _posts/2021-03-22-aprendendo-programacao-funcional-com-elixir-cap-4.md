---
layout: post
title:  Aprendendo programação funcional com elixir - cap4
date:   2021-03-22 19:30:00 -0300
image:  functional01.jpg
tags:   Funcional, Elixir
---

# Conhecimentos prévios

* Listas - Head e tail

* Como acessar os valores de um map (tem resumo nesse cap mesmo, caso não queira ir atrás)

* Controle de fluxo com Pattern Match

# Intro

* Funções recursivas são o core de repetições em programação funcional

* Em linguagens imperativas nós usamos for e while, que são baseados em modificar o estado

* PORÉM na linguagem funcional, nós temos dados imutaveis, então a gente usa outra abordagem

# Surronded by Boundaries

* Podemos ter uma recursão que nós conseguimos falar quando ela acaba, e tem outras que são infinitas. Nesse pequeno módulo vamos lidar com a primeira (bounded recursion, ou recursão com limite...?)

* Nesse tipo de recursão nós estamos fazendo uma iteração por vez, mas cada vez que chamamos a próxima iteração, estamos reduzindo os valores passados pra ela até chegarmo em um ponto de parada

* (ex: recursao([1,2,3]) -> próxima iteração -> recursao[1,2] -> próxima iteração -> recursao[1] -> próxima iteração -> recursao[] -> Parou (pensando que essa é a clausula de parada))

* O número de iterações em uma recursão tem relação direta com o valor do seu argumento (no exemplo acima é uma lsita de 3 itens (+ uma lista vazia), então ele vai rodar 4 vezes )

```elixir
defmodule Sum do
  def up_to(0), do: 0 # Clausula de parada - vamos ver mais pra frente
  def up_to(n), do: n + up_to(n-1)
end
######################
up_to(3)
= 3 + up_to(2) 
= 3 + 2 + up_to(1)
= 3 + 2 + 1 + up_to(0)
= 3 + 2 + 1 + 0
= 6
```

* No final das contas recursão é chamar a mesma função repetidamente, diminuindo o argumento, de forma que ele chegue em uma clausula de parada
  
* Caso não tenha clausula de parada, ele vai entrar em um loop infinito e travar tua aplicação

## Navigating Through Lists

* Vamos precisar iterar por listas ao longo da nossa carreira programando
  
* E elixir a melhor forma de navegar por listas usando recursão é através de `[head|tail]`
  
* Relembrando aqui como funciona o head e tail:
  * Em uma lista head é o primeiro elemento de uma lista, enquanto tail é uma lista com os elementos que sobraram (ou uma lista vazia, no caso de ser uma lsita de um único item)
  * Vai dar par entender um pouco melhor no exemplo abaixo (que é similar ao exemplo acima, mas no lugar de usarmos integer, nós usamos uma lista)
  
```elixir
defmodule Matematica do
  def soma([]), do: 0 # Clausula de parada - vamos ver mais pra frente
  def soma([head|tail]), do: head + up_to(tail)
end
#########
soma([10, 20, 30])
= 10 + soma([20, 30])
= 10 + 20 + soma([30])
= 10 + 20 + 30 + soma([])
= 10 + 20 + 30 + 0
= 60
```

* Da pra ver nesse exemplo que a cada iteração a gente vai chamando a função soma passando uma lista menor, até chegar ao ponto que a lsita está vazia e chegamos a nossa clausula de parada.

### Conteudo bonus por Rachel :P

* A coisa mais importante que nós temos em uma recursão é a clausula de parada

* Para identificar a clausula de parada nós devemos nos perguntar "se eu passar o menor valor nessa função, qual o comportamento que eu espero que aconteça?" (no caso de listas o menor valor é `[]` no caso de numero geralmente é `0`)

* O que eu entendi é que a primeira coisa que você precisa definir é a clausula de parada
  
* e que a partir do momento que você descobre como a primeira iteração funciona, da pra substituir parte do problema pela recursão E SÓ CONFIAR QUE ELA VAI FUNCIONAR

* Um exemplo simples: Somar os numeros de uma lista.
  1. A primeira coisa que você deve fazer é se perguntar "mas se eu passar uma lista vazia. O que acontece?
  2. A segunda coisa é escrever o exemplo mais simples. Se a lista tiver um numero só?
  3. depois você vai pra uma iteração mais complexa: o que acontece se a lista tem 2 numeros?
  4. Com esses três exemplos você geralmente consegue entender qual a lógica por trás da sua função e substituir parte da sua função de 2 numeros pela recursão. (muitas vezes da pra usar só os dois primeiros exemplos, mas o terceiro te ajuda a entender)

```elixir
# Exemplos para entender melhor como vai funcionar nossa função (coloquei a lista vazia no final das listas para ilustrar melhor)
def soma([]), do: 0 # 1
def soma([n, []]), do: n + 0 # 2 (vamos refatorar aqui depois)
def soma([n, m, []]), do: n + m + 0 # Entendendo o caso mais simples da recursão. Vai ser limado na refatoração

# Refatoração pra implementar recursão:
def soma([]), do: 0 # caso base :D
def soma([head | tail]), do: head + tail # sabendo que tail é uma lista (mesmo que vazia), você ja pode embrulhar ela na sua função de soma de lista e CONFIAR na recursão.

# Resultado final:
def soma([]), do: 0 # caso base :D
def soma([head | tail]), do: head + soma(tail)
```

## Transforming lists

* No dia a dia nós sempre esbarramos com a necessidade de transformar listas. As vezes recebemos uma lista de usuários e precisamos retornar todas as cidades deles agrupadas por estado, ou precisamos filtrar quais são os usuários maiores de idade (e por ai vai)

* Sendo que na programação funcional, dados são imutáveis, então não temos como fazer essas transformações (oh no). Quando estamos iterando por uma lista e transformando os valores delas, na verdade nós estamos criando novas listas :D

* A sintaxe de head e tail não apenas é util para desconstruir uma lista, mas também é útil para construir novas listas

* Nós vimos anteriormente que conseguimos concatenar duas listas usando o operador `++` (`[1, 2] ++ [3,4] == [1,2,3,4]`). Os elementos da primeira lista vem primeiro, e estamos dando um append na segunda lista de forma que os elementos dela vem depois.

* Mas podemos adicionar elementos em uma lista usando head e tail :D

```elixir
 lista1 = [1, 2]
 lista2 = [3 | lista1] # isso vai ser igual a [3, 1, 2]
```

* Vale lembrar que uma lista `head` é igual a um elemento enquanto `tail` é sempre é uma lista.

```elixir
 lista1 = [1, 2]
 lista3 = [[3,4] | lista1] # isso vai ser igual a [[3,4] 1, 2]
```

* Se quisermos concatenar duas lista usando `head` e `tail`, usariamos recrusão (exemplo a seguir)
* Mas qual a vantagem de criar uma lista com `[head|tail]` ao invés de usar `++`?

  * performance :D

* Antes de continuarmos nos exemplos, vamos fazer uma revisão rápida sobre como acessar os itens dentro de um map

### Key based accessor

```elixir
bolo = %{sabor: "Bolo de limão com cobertura", preco: 20, cobertura: true}
bolo[:sabor] #vai retornar limão
bolo[:confeitero] #vai retorna nil <---- !!!!! importantee

###### 
bolo.sabor #vai retornar limão
bolo.confeitero # va estourar erro <---- !!!!! importantee
```

### Exemplo :D

* Vamos supor que você é um confeiteiro que faz bolos.

```elixir
bolos = 
  [
   %{sabor: "bolo de limão", preco: 15, cobertura: true, pronto: false},
   %{sabor: "bolo simples de laranja", preco: 15, cobertura: false, pronto: true},
   %{sabor: "bolo de banana", preco: 25, cobertura: false, pronto: true},
   %{sabor: "bolo de chocolate", preco: 20, cobertura: true, pronto: false},
  ]
```

* Cada bolo tem um nome, o preço e uma tag `cobertura` para sabermos se ele vai precisar de cobertura, além de uma tag que informa que ele está pronto pra venda

* Você recebe uma lista de bolos, coloca cobertura nos que precisam de cobertura, atualiza o nome, o preço (como a cobertura é muuuuito boa, o valor dos bolos vai dobar) e informa que ele está pronto pra venda

```elixir
def bolos_para_cobrir([]), do: []
def bolos_para_cobrir([ bolo | proximos_bolos]) do:
  novo_bolo = %{
    sabor: "#{bolo.sabor} com cobertura"
    preco: bolo.preco * 2
    cobertura: true,
    pronto: true
  }
  
  [novo_bolo | bolos_para_cobrir(proximos_bolos)]
end
```

* o resultado é o seguinte:

```elixir
novos_bolos = bolos_para_cobrir(bolos)
novos_bolos == 
  [
   %{sabor: "bolo de limão com cobertura", preco: 30, cobertura: true, pronto: true},
   %{sabor: "bolo simples de laranja com cobertura", preco: 30, cobertura: false, pronto: true},
   %{sabor: "bolo de banana com cobertura", preco: 50, cobertura: false, pronto: true},
   %{sabor: "bolo de chocolate com cobertura", preco: 40, cobertura: true, pronto: true},
  ]
```

* mas qué que ta acontecendo?

  1. chamamos a função `bolos_para_cobrir` passando os bolos que precisavamos cobrir
  2. Ele vai olhar pra primeira claucula e perguntar "os argumentos que nós estamos passando é igual a `[]`"
  3. Como a resposta é não, ele vai passar pra segunda função, criar um bolo com cobertura, e colocar no head de uma nova lista
  4. O tail dessa lista vai ser uma nova chamada pra função `bolos_para_cobrir`

* no final das contas esse é o resultado (vou simplifcar os sabores chamado cada um de `saborn` e vou ilustrar que no final de cada lista tem uma lista vazia)

```elixir
[sabor1_m | bolos_para_cobrir([sabor2, sabor3, sabor4, []])]
[sabor1_m, sabor2_m | bolos_para_cobrir([sabor3, sabor4, []])]
[sabor1_m, sabor2_m, sabor3_m | bolos_para_cobrir([sabor4, []])]
[sabor1_m, sabor2_m, sabor3_m, sabor 4_m | bolos_para_cobrir([[]])]
[sabor1_m, sabor2_m, sabor3_m, sabor 4_m | [] ]
[sabor1_m, sabor2_m, sabor3_m, sabor 4_m]
```

* MAS POR QUE TODOS OS BOLOS TEM COBERTURA, MESMO OS QUE JÁ ESTAVAM PRONTOS E NÃO PRCISAVA DE COBERTURA?

  * Por que a gente em nenhum momento perguntou se tava pronto

    * E como fazemos isso? com if?

      * Naaaao! Usando controle de fluxo via pattern match :D (mas pode ser com if também, mas assim é mais legal)

* Agora a gente vai adicionar mais uma função `bolos_para_cobrir` para lidar com os bolos que a gente não quer cobrir

```elixir
def bolos_para_cobrir([]), do: []
##### ESSA FUNÇÃO ABAIXO É NOVA :D
def bolos_para_cobrir([bolo = %{pronto: true} | proximos_bolos]) do
  [bolo | bolos_para_cobrir(proximos_bolos)]
end
#### CABO FUNÇÃO NOVA
def bolos_para_cobrir([ bolo | proximos_bolos]) do:
  novo_bolo = %{
    sabor: "#{bolo.sabor} com cobertura"
    preco: bolo.preco * 2
    cobertura: true,
    pronto: true
  }
  
  [novo_bolo | bolos_para_cobrir(proximos_bolos)]
end
```

* E o resultado vai ser esse:

```elixir
novos_bolos = bolos_para_cobrir(bolos)
novos_bolos == 
  [
   %{sabor: "bolo de limão com cobertura", preco: 30, cobertura: true, pronto: true},
   %{sabor: "bolo simples de laranja", preco: 15, cobertura: false, pronto: true},
   %{sabor: "bolo de banana", preco: 25, cobertura: false, pronto: true},
   %{sabor: "bolo de chocolate com cobertura", preco: 40, cobertura: true, pronto: true},
  ]
```

* Qué que ta acontecendo?

  1. Assim como da ultima vez, nós chamamos a função `bolos_para_cobrir` passando nossos bolos

  2. Ele vai passar pelo primeiro pattern match. `bolo == []`? a resposta é não, então ele vai pra segunda verificação

  3. o primeiro item (`head`) da nossa lista de bolos tem a chave `pronto: true` a resposta é não, então ele passa pra terceira função, que é mais genérica ( é meio que nosso else)

  4. O nosso bolo de limão faz os requisitos que a gente estipulou pra receber a cobertura, então a gente coloca ele no começo da nova lista de bolos e mais uma vez chamamos a função.
  5. Nossa proxma lista é `[laranja, banana, chocolate]`, e ela não atende o primeiro patteren match (`[]`)

  6. mas atende o segundo, já que o primeiro item da nova lista é um bolo de laranja que tem a chave `pronto: true` 

  7. Então ele vai adicionar o bolo de laranja na lista, sem alterar em nada

  8. E assim ele vai fazer esse movimento até a lista ficar vazia e atingir nossa clausula de parada

# Conquering Recursion

* Aprender recursão pode ser um pouco chatinho

* O ulisses compilou duas técnicas: decrease and Conquer (diminuir e conquistar) e divide and conquer (dividir e conquistar)

* Rachel: eu aprendi recursão no curso "how to code data" - simple e complex. É basicamente Decrease end Conquer, mas a abordagem do curso é como lidar com tipos de dados diferentes (e consequentemente aprendemos de uma forma mais natural a lidar com recursão)

## Decrease and Conquer

* Diminuir e conquistar consiste em diminuir o problema na sua forma mais simples

* Primeiro tentamos descobrir qual a solução mais óbvia para um pedaço minúsculo do problema

* depois é incrementar o problema um pedaço por vez

* Para ver essa abordagem, usaremos fatorial (ex: fatorial de 3 é `3 * 2 * 1`):

### Exemplo

* Queremos uma função que calcule o fatorial do numero que passaremos como parametro

1. O primeiro passo é tentar descobrir qual é o caso de base.

    * Qual a nossa clausula de parada?

    * Qual é o caso mais simples que nossa função vai receber?

1. Para nos ajudar a achar o caso base vale tentar fazer a função na mão com uma parte pequena do problema

```elixir
defmodule Factorial do
  def of(0), do: 1
  def of(1), do: 1
  def of(2), do: 2 * 1
  def of(3), do: 3 * 2 * 1
  def of(4), do: 4 * 3 * 2 * 1
end
```

* Se a gente for destrichar o código acima para transformar em uma função, nós temos:

* `def of(1)` é exatamente igual a `1 * of(0)`

* e `def of(2)` é igual a  `2 * of(1)`

* indo na mesma linha `def of(3)` é exatamente igual a `3 * of(2)` (e por ai vai)

* Então se analisarmos com calma, nossa função recursiva é igual a

```elixir
def of(n), do: n * of(n - 1)
```

* e está pronto a sua recursão :D

## Divide and Conquer

* Dividir e Conquistar é uma técnica sobre separar o problema em partes menores que podem ser processadas de forma independente

* (Essa tarefa não funciona apenas para algoritimos, mas na vida ela também é bem útil. Se não sabe como começar um projeto ou uma tarefa nova, vale dividir em pedaços menores e atacar um por vez)

* Essa técnica é bem interessante para quando quisermos organizar elementos

* No exemplo acima nós focamos em destrinchar 1 função, mas nesse exemplo nós vamos usar 2:

    1. Uma para dividir

    2. Uma função para resolver o nosso problema (no exemplo a seguir iremos usar uma função que organiza itens de uma lista em ordem crescente)

### Exemplo - intro

* No exemplo vamos escrever uma função que recebe uma lista e organiza ela em ordem crescente

* O ideal é dividir essa lista no menor pedaço possível (no caso listas com apenas 1 elemento) e depois juntar essas listas de forma ordenada

### Exemplo - parte 1 - Dividir

* Para dividir uma lista em duas nós usamos `Enum.split/2`
  
  * O primeiro argumento é uma lista e o segundo é a quantidade de elementos que você quer que tenha na primeira lista (pro-tip: abrir o iex -> `h Enum.split` para entender melhor o que faz o `Enum.split`. O `h` na frente da função significa `help`)
  
  * `Enum.split([:a, :b, :c, :d], 1) # => {[:a], [:b,:c,:d]}`

* O problema é que queremos dividir a lista no meio, mas não sabemos quantos elementos tem na lista

  * Para sabermos quantos elementos tem na lista, nós podemos dar um `Enum.count(lista)`

  * E para dividirmos em dois, o melhor é usar a função `div/2` (que o primeiro argumento é o numero que vocë quer dividir, e o segundo é por quanto)

    * Porque usar o `div` e não o `/`?

    * Porque quando dividimos um numero impar usando `/` nós temos um float (`3/2 == 1.5`) enquanto a função `div` vai retornar um inteiro (`div(3,2) == 1`)

    * E queremos um inteiro porque a função `Enum.split` recebe um inteiro como segundo agumento

* OKEI! VOLTANDO AQUI A PEGAR ESSE CONHECIMENTO E APLICAR EM UMA FUNÇÃO QUE DIVIDE UMA LISTA EM 2

* Agora que já sabemos como dividir uma lista em duas, podemos construir a primeira parte da função (estou colocando uns asteriscos numerados pra comentar depois coisas individuais sobre as linhas marcadas)

```elixir
defmodule Sort do
  def ascending([]), do: [] # *1
  def ascending([a]), do: [a] # *1
  def ascending(list) do
    half_size = div(Enum.count(list), 2) # *2
    {list_a, list_b} = Enum.split(list, half_size) #*3 e #*4
    
    # Agora que ja temos s listas divididas, nós precisamos organizar
    # as duas listas de forma ascendente :D Isso vai pra parte 2 do exemplo
  end
end
```

1. Vale atentar que nós temos duas clausulas de parada nessa função. O que acontece se passarmos uma lista vazia pra ser organizada? mas se passarmos uma lista com elementos, qual o menor pedaço dela?

2. A gente ainda não viu pipe, mas aqui é um bom lugar pra ir um pipe :x Fica essa observação pra pessoa programadora do futuro

3. Quando usamos o `Enum.split\2` a assinatura dele é uma tupla com duas lisstas dentro (usando o `h Enum.split` no terminal a gente consegue ter essa certeza)

4. Estamos fazendo pattern match com o resultado do `Enum.split` e criando duas variáveis para cada lista formada (pura magia!)

### Exemplo - parte 2 - Organizar

* Agora precisamos criar uma função que receba duas listas e crie uma nova de forma organizada

* a idéia é que se ele recebe `[9]` e `[5]`, ele retorne uma lista `[5,9]`

* vamos ver criar uma função chamada merge e entender como funciona o passo a passo de juntarmos a lista `[5,9]` com `[1,4,5]` de forma recursiva

```elixir
merge([5,9], [1, 4, 5])
[1 | merge([5,9], [4, 5])]
[1, 4 | merge([5,9], [5])]
[1, 4 | merge([5,9], [5])]
[1, 4, 5 | merge([9], [5])]
[1, 4, 5, 5 | merge([9], [])]
[1, 4, 5, 5, 9]
```

* PS: Só relembrando que para esse mege funcionar, nós precisamos que a lista já esteja ordenada. Por iso que estamos tendo todo o trampo de dividir na menor parte possível - pra garantir que ela esteja ordenada

* Agora que entendemos como a função de merge deve funcionar, vamos escrever ela

```elixir
defp merge(list_a, []), do: list_a # * 1
defp merge([], list_b), do: list_b # * 1
defp merge([head_a|tail_a], list_b = [head_b | _]) when head_a <= head_b do
  [head_a | merge(tail_a, list_b)] # * 2
end
defp merge(list_a = [head_a| _], [head_b | tail_b]) when head_a > head_b do
  [head_b | merge(list_a, tail_b)] # * 2
end
```

1. As duas primeiras linhas sáo nossa clausula de parada. Como estamos diminuindo a lista acada iteração, em algum dado momento uam das listas vai ficar vazia. E o que queremos que aconteça no caso de uma lista vazia? Que retorne a que não está vazia

    * Vale a obs que essa clausula também cobre o caso de ambas as listas estarem vazias, já que tanto `list_a` quanto `list_b` podem ser listas vazias também.

2. Aqui que a magia acontece. Nós estamos recebendo duas listas(já pré-ordenadas), comparando os primeiros itens de cada uma, e criando uma nova lista :D

### Exemplo - parte 3 - Juntando tudo

* Okei. Nós temos uma função que divide uma lista e outra que junta listas ordenando elas. Como juntar?

```elixir
defmodule Sort do
  def ascending([]), do: [] # *1
  def ascending([a]), do: [a] # *1
  def ascending(list) do
    half_size = div(Enum.count(list), 2) # *2
    {list_a, list_b} = Enum.split(list, half_size) #*3 e #*4

    ##### JUNTAMOS AS DUAS AQUI
    merge(
     ascending(list_a),
     ascending(list_b)
    )
  end
  
  defp merge(list_a, []), do: list_a # * 1
  defp merge([], list_b), do: list_b # * 1
  defp merge([head_a|tail_a], list_b = [head_b | _]) when head_a <= head_b do
    [head_a | merge(tail_a, list_b)] # * 2
  end
  defp merge(list_a = [head_a| _], [head_b | tail_b]) when head_a > head_b do
    [head_b | merge(list_a, tail_b)] # * 2
  end
end
```

* qué que ta acontecendo? Nós estamos fazendo uma recursão dentro de outra?

  * Sim!

* Vamos confiar que a nossa função ascending vai pegar uma lista e dividir ela em listas menores até chegar em uma lista com um único elemento, certo? (você vai passar uma lista, e ela divide ela em duas. passaremos uma nova lista, e ela irá dividir em duas até chegar na clausula de parada, que é `[]` ou `[a]` )

* quando chamamos a função merge em uma lista `[9, 5, 1, 5, 4]` o que irá acontecer é o seguinte

```elixir
merge(merge([9], [5]), merge(merge([1], [5], [4])))
merge([5,9], merge(merge([1], [5], [4])))
merge([5,9], merge([1, 5], [4]))
merge([5,9], [1, 5 ,4]))
```

* Pode parecer confuso, mas o que ele ta fazendo nada mais é do que ir dividindo a lista no menor tamanho possível e mergear duas listas com 1 elemento cada.

* NO FINAL DAS CONTAS

* divide e conquer nada mais é do que você dividir o problema em partes mastigaveis, e atacar cada parte dela com o decrease e conquer

# Tail Call Optimzaton (TCO)

* Quando nós chamamos uma função, isso consome memória e gerlmente isso não é um problema

* mas quando nós estamos chamando milhões de funções recursivas, isso é um problema

* Acho que é interessante entender que uma função consome memória enquanto ela está sendo processada

* se eu chamar função de soma, por ex, o que vai acontecer é o seguinte

  * Chamei a função soma -> processa -> (ocupa espaço na memória enquanto está processando) -> termina de processar -> exibe o resultado (desocupa o espaço na memória)

* Quando estamos fazendo recursão o que acontece é o seguinte:
  * Chamei função recursiva que ainda não chegou na clausula de parada -> (ocupa espaço na memória) -> Chamo a mesma função recursivamente -> (aloco outro espaço na memória pra essa segunda chamada) -> Chamo mais uma vez -> (mais um tereiro espaço na memória) -> (...) -> chego na clausula de parada -> termino esse procesamento -> desocupo UM espaço na memória -> passo o resultado pra frente (e assim vou desalocando a memória uma função por vez)

* Dai quando estamos fazendo um fatorial de 3, 4 é tranquilo, mas quando quisermos fazer um fatorial de 10.000.000 a história já muda

* MAS COMO VAMOS RESOLVER ESSE PROBLEMA?

* Algumas linguagens (como elixir) tem o recurso de tail call optimization, que é uma otimização que acontece quando a ultima linha do seu código é a chamadada para sua função recursiva

* Nos exemplos acima (factorial, assending e merge) nenhuma das funções está otimizada. A ultima linha da função factorial é `n * of(n-1)`, na função de asseding é `merge(assending(x), assending(y)` e na função de merge é `[head_x| merge(list_y, tail_x)]`

* muitas vezes para você fazer a sua função ser otimizada, é preciso repensar a função e colocar um parametro auxiliar

* Vamos dar o exemplo usando fatorial

```elixir
def of(n), do: factorial_of(n, 1) # *1
defp factorial_of(0, acc), do: acc # *2
defp factorial_of(n, acc), do: factorial_of(n-1, n * acc) # *3
```

* O que ta acontecendo linha por linha:

1. Ao chamar a função `of`, nós estamos chamando a função auxiliar `factorial_of`  passando n, e um acumulador (chamos ela de função trampolim. Ela vai dar o pontapé inicial ao calculo do fatorial)

    * O acumulador tem ligação direta com a clausula de parada que tinhamos antes. O que queremos que aconteça quando passamos o menor valor? nesse caso o menor valor é 0, e se passarmos 0 queremos que o resultado da nossa função seja 2

2. Cuidamos da nossa clausula deparada da função auxiliar que nós criamos para otimização

3. O processamento é feito direto na chamada da função. Dessa forma essa função será processada quando for chamada e não ficará ocupando espaço na memória :D

    1. também funcionaria se essa função fosse escrita como:

    ```elixir
    defp factorial_of(n, acc) do
      novo_acumulado = n * acc
      factorial_of(n-1, novo_acumulado)
    end
    ```

* mas nem tudo são flores

* Tail call optimization é bom porque diminuiu a chance de você ter um problema de stack overflow.

* Porém a legibilidade pode ficar bastante comprometida

```elixir
### mais legivel
def of(n), do: n * of(n - 1)

### Menos legível
def of(n), do: factorial_of(n, 1)
defp factorial_of(0, acc), do: acc
defp factorial_of(n, acc), do: factorial_of(n-1, n * acc)
```

* Então na hora de fazer sua recursão, é importante colocar isso na balança. Eu reeeealmente preciso otimizar minha função? Sou to time que sempre que puder escolha legibilidade (mas as vezes não da, né?)

# Functions Without Borders

* As vezes temos que escrever recursão sem sabermos qual a clausula de parada

* exemplo disso são webcrawling ou quando precisamos passear por uma arvore de arquivos

* Cada página que nós passamos, não sabemos quantas mais vem pela frente

* O maior cuidado que precisamos ter é o de evitar looping infinito.

## Adding boundaries

* A primeira forma que temos para evitar looping infinito é o de adicionar um limite.

* Podemos adicionar um contador sobre qual a "profundidade" que estamos entrando e, quando chegar nesse limite nós paramos de procesar

* (ex: recebemos uma lista de páginas e processamos as páginas. Essas páginas tem links, e a gente quer processar a página desses links também. Faremos isso até chegar em uma "profundidade" que nós estipulamos)

## Avoiding Infnite Loops

* É sempre interessante monitorar quais são as possiveis causas de um loop infinito.

* Nos exemplos acima, por exemplo podemos ter links simbolicos dentro das pastas que levam para pastas que já processamos (e assim criar um loop infinito) ou no caso de páginas da web, podemos ter links para páginas já processadas

* Muitas vezes adicionar limite resolve o problema, mas podemos fazer uma verificação adicional

  * vamos supor que você esteja armazenando as páginas no seu banco de dados. As vezes vale ver se ela existe antes de salvar

  * Se o é de buscar nas pastas do sistema, vale verificar se é um diretório válido antes de salvar

# Using Recursion with anonymous Functions

* é possível, mas de forma geral não vale a pena. Não tem 10% da legibilidade de uma função nomeada

* mas se você reeeealmente quiser fazer, aqui vai um exemplo:

```elixir
iex> factorial_gen = fn me ->
  fn
    0 -> 1
    x when x > 0 -> x * me.(me).(x-1)
  end
end

iex> factorial = factorial_gen(factorial_gen)
iex> factorial.(5)
120
```
