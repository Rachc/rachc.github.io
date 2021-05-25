---
layout: post
title:  Aprendendo programação funcional com elixir - cap3
date:   2021-03-20 19:30:00 -0300
image:  functional01.jpg
tags:   Funcional, Elixir
---
# Disclaimer
* Os posts dessa série são minhas anotações pessoais dos meus estudos do livro "Aprendendo programação funcional com elixir.

* Eles não passaram por revisão, possuem typos e alguns erros. 

* Estão mais próximos de pensamentos desconexos do que de um texto estruturado.

* Dito isso, segue anotações (: 

# Intro ao capitulo

* Nas linguagens imperativas `if` é o nosso principal controlador de fluxo

* nas linguagens funcionais `pattern match` é o que usamos mais

# Making two things match

* as utilidades do pattern matching são várias:

  * Acessar variáveis

  * acessar valores

  * tomar decisões sobre o tipo de função que será invocada

* O operador de pattern match é o `=` (sim, é o mesmo que faz a associação de variáveis e valores)

* Caso um valor não der match com o outro, ele exibe o erro `matchError`, parando a execução do programa (por outro lado, se der bom o programa vai continuar rodando)

* A melhor forma de aprender é testando no terminal :D (como quase tudo).

```elixir
iex> 1 = 1
1
iex> 2 = 1
** (MatchError) no match of right hand side value: 1
iex> 1 = 2
** (MatchError) no match of right hand side value: 2
```

* Agora vamos fazer a mesma coisa com variáveis

```elixir
iex> x = 1 # primeiro atribuimos o valor 1 a variável
1
iex> 1 = x
1
iex> 2 = x
** (MatchError) no match of right hand side value: 1
```

* A primeira vista pode parecer meio bobo. Pra que isso serve?

* mas se pensarmos como um if, talvez faça mais sentido. O que o elixir está tentando fazer é isso aqui:

```elixir
if 2 == x do
  2
else
  raise MatchError
end
```

* E o que acontece quando invertemos de `2 = x` para `x = 2`? Ele vai dar uma nova atribuição a variável `x`

* Então pensa assim: Quando você usa `=` ele tenta fazer um match. Se a variável está do lado esquerdo (`x = 2`) ele tenta fazer uma associação. Caso ela esteja do lado direito (`2 = x`) ele faz apenas o match

* E se quisermos usar a variável na esquerda sem ter que fazer binding?

* usamos o `pin operator` (`^`).

* Se quisermos usar comparar `x = 2` sem que tenhamos que fazer a associação, fazemos `^x = 2` ( Rachel: pode não parecer util agora, sempre que eu preciso fazer uma query no ecto - uma lib elixir que se comunica com o banco de dados eu preciso fazer esses binding. É como se eu dissesse "amigo, eu só quero procurar por `x`, me deixa")

* Caso ainda esteja dificil de  entender, da pra comparar com aritimetica. "se `x = 1`, então `1 = x`"

# Unpacking Values from various data type

* Pattern match também é bastante útil para extrair valores de diferentes tipos de dados

* (Rachel: Uma das minhas coisas favoritas de pattern match <3)

* Imagina que você recebeu um enorm map e precisa de um ou 2 valores: Da pra extrair isso com pattern match (e é lindo)

## Matching Parts of a string

* É possível fazer pattern match em uma string ao mesmo tempo que salvamos ela em uma variável se usarmos o operador `<>`

```elixir
iex> "Authentication: " <> credentials = "Authentication: Basic 123456789"
iex> credentials
"Basic 123456789"
```

* Omeudeus, o qué que ta acontecendo ai?!?!?!

* Vamos pensar que o elixir está comparando duas expressões. A primeira é `"Authentication: " <> credentials` enquanto a segunda é `"Authentication: Basic 123456789"`

* Se a segunda expressão começar com `"Authentication: "`, ele vai salvar tudo que vier depois na variável `credentials`

* Então ao mesmo tempo que ele verifica se o lado direito é igual ao esquerdo, ele guarda o valor que precisamos em variáveis! (e isso é top!)

* Algo importante a ser levando em consideração ao comparar se usar `<>` e `=` nas strings:

  * o `<>` tem que vir primeiro do que o `=`

```elixir
iex> "Authentication: " <> credentials = "Authentication: Basic 123456789"
#da bom
iex> "Authentication: Basic 123456789" = "Authentication: " <> credentials
** (CompileError) a binary field without size is only allowed at the end of a binary pattern and never allowed in binary generators
# da ruim
```

* (ps: em elixir String e binários são a mesma coisa. Na real em erlang, Strings são chamados de "binários")

## Matching Tuples

* Tuplas são coleções que são armazenadas continuamente na memória, de forma que você consegue acessar seus valores diretamente pelo index

* São muito válidas para passar um "sinal" com valores - ex: `{:ok, "mensagem"}` (o `:ok` é um atom que serve pra sinalizar que deu bom, e a mensagem pode ser o valor de uma operação)

* nós podemos acessar itens de uma tupla e associa-los a uma variável com uma simples expressão:

```elixir
iex> {a, b, c} = { 1, 2, 3}
iex> a
1
iex> b
2
iex> c
3
```

* também podemos usar tuplas pra fazer pattern match e definir o fluxo da nossa aplicação apenas quando for um caso de sucesso (vamos ver isso a fundo logo mais, mas até chegar lá fica uma degustação)

```elixir
processe_a_vida_o_universo_e_tudo_mais = fn -> {:ok, 42} end

iex> {:ok, resposta} = processe_a_vida_o_universo_e_tudo_mais.()
iex> IO.puts "A resposta é #{resposta}."
A resposta é 42
```

* O QUE QUE ACONTECEU AI?

  1. Nós criamos uma função que processa a vida, o universo e tudo mais, e o seu retorno é `{:ok, 42}`

  2. Na linha abaixo nós stamos salvando chamando a função que processa a vida o universo e tudo mais (do lado direito) (cujo retorno é `{:ok, 42}` e fazendo o pattern match do lado esquerdo

  3. SE o retorno da função da direita for uma tupla cujo o primeiro elemento é `:ok`, a gente vai salvar o segundo elemento em uma variável chamada `resposta`

  4. para testar se isso deu bom, a gente mandou um IO.puts usando a resposta em uma frase

  5. (se o retorno da função fosse diferente de `{:ok, _}`, o programa ia quebrar miseravelmente)

## Matching Lists

* Aprendemos que tupla é uma coleção de vários itens, mas tupla possui um problema

* Nós precisamos saber de antemão quantos elementos tem dentro delas (mas nem sempre conseguimos saber disso)

* Para lidar com esse problema, nós temos listas :D

* Listas são guardadas na memória de forma linkada. Se você quiser ter acesso ao terceiro item da sua lista, por exemplo, você precisa passar sempre pelo primeiro e pelo segundo.

* Vale frizar que o final de uma lista linkada é sempre uma lista vazia (na prática isso não é muito importante, mas saber disso ajuda quando formos pensar em recursão)

* Assim como acontece com tuplas, também conseguimos acessar os valores dentro de listas através de pattern match, além de salvar esses valores em variaveis

* A sintaxe de listas é `[]` (com os elementinhos dentro `[1,2,3]`)

```elixir
[a,a,a] = [1,1,1] # Vai dar match, pois estamos procurando uma lista de 3 itens que possui os mesmos valores
[a,a,a] = [1,2,1] # Não vai dar match, pois os itens não são iguais
[a,b,a] = [1,2,1] # top!
```

* Podemos ser mais espcíficos com o nosso pattern match

```elixir
[a, a, "banana"] = ["abacaxi", "abacaxi", "banana"] # vai dar match, pois estamos procurando por uma lista que possua 3 itens, sendo os dois primeiros iguais e o ultimo PRECISA ser "banana"
```

### Caracter _ no pattern match

* Também podemos ser mais vagos no nosso pattern match

```elixir
[_, a, _] = [1,5,23] # nesse caso procuramos uma lista de 3 elementos, e eu estou interessada APENAS no segundo.
```

* O caracter `_` significa que não me importa o valor desse item na lista. Pode vir qualquer coisa, que da match

### head | tail

* Até então todos os exemplos que nós mostramos tem ligação direta com o tamanho da lista, porém a graça da lista é que não precisamos saber do seu tamanho.

* Então como lidar?

* Em elixir nós podemos usar o operador `|` para ter acesso aos itens de uma lista que não sabemos o tamanho

* Comoassim?

```elixir
iex> [head|tail] = [1, 2, 3, 4, 5]
iex> head
1
iex> tail
[2, 3, 4, 5]
```

* No exemplo acima nós usamos o head para ter acesso ao primeiro valor de uma lista, e tail é uma lista com o restante dos valores

* Usamos `head` e `tail` como variaveis padrão, mas pode ser qualquer coisa

* no exemplo acima poderia muito bem ser `[numero | lista_com_o_resto_dos_numeros]` (ou apenas `resto` :P)

* O lado esquerdo do operador `|` vai retornar valores, enquanto o lado direito vai retornar uma **lista** de valores.

* Porque eu to falando isso? Porque é uma introdução para falar que do lado esquerdo podmos retornar mais de um item

```elixir
iex> [a, b | resto] = [1, 2, 3, 4, 5]
iex> a
1
iex> b
2
iex> resto
[3, 4, 6]
```

* Agora lembra quando eu falei acima que no final de toda lista tem uma lista vazia? Saber disso é bem útil se formos pensar em uma lista de um único valor

```elixir
iex> [head | tail] = ["pastel"]
iex> head
"pastel"
iex> tail
[]
```

* Em contrapartida se usarmos a sintaxe de  `[head | tail]` em uma lista vazia, nós vamos receber um erro na cara

* MAS POR QUE C TA ME FALANDO ISSO, RACHEL?

* Porque esse conhecimento é muito útil pro caso de estarmos precisarmos saber se a lista só tem um unico elemento

* Imagina que estamos iterando por uma lista. Em algum momento nós precisamos saber se aquele elemento é o último da lista.

## Matching Maps

* Mapas são um tipo de dado estruturado no conceito de chave e valor

* Por exemplo, se quisermos representar um usuário fazendo um login, podemos guardar os valores que ele nos envia em um mapa

```elixir
login_do_usuario = %{email: "rachel@email.com", senha: "123456"}
```

* A sintaxe de `%{}` ée o que defin eum mapa

* Quando usamos chaves e valores dessa forma: `email: "rachel@email.com"` significa que setamos usando um atom como chave

* Não precisamos ficar limitados com chaves no formato de atom. Podemos usar qualquer coisa. A diferença é como nós vamos representar esses valores em um map (e como acessar os valores dentro do map. mas chegaremos já ai)

* Vamos usar dessa vez a sintaxe `=>`:

```elixir
vendas = %{"2020/1" => 2000, "2020/2" => 2500}
```

* No exemplo acima nós temos uma string como chave :D

* Vale uma observação que um map representado como `%{chave: "valor"}` nada mais é do que `%{:chave => "valor"}`, mas preferimos usar a primeira forma

* Conseguimos aninhar estruturas complexas dentro de um mapa também

```elixir
%{
  tem: "Pastel",
  quantdade: 10,
  sabores: ["queijo", "carne", "frango"],
  outras_infos: %{chave: "valor", chave2: "valor2"} #fiquei sem criatividade aqui. Malz ae
}
```

### Acessando um item dentro de um mapa (conteudo bonus)

* (No livro isso é visto no capitulo seguinte, mas já que estamos falando sobre mapas agora, nada mais justo do que falar sobre isso aqui)

* A forma mais universal de se acessar um map é `mapa[:chave]` (ou `mapa["chave"]`. isso depende do tipo de sua chave)

* porém se temos um map que usa a sintaxe `mapa = %{chave: valor}` ele pode ser acessado como `mapa.chave`

* A forma que você chama o elemento de um map, caso não exista aquela chave, também vai influenciar no retorno:

```elixir
bolo = %{sabor: "Bolo de limão com cobertura", preco: 20, cobertura: true}
bolo[:sabor] #vai retornar limão
bolo[:confeitero] #vai retorna nil <---- !!!!! importantee

###### 
bolo.sabor #vai retornar limão
bolo.confeitero # va estourar erro <---- !!!!! importantee
```

### Okei, finalmente pattern match :D

```elixir
bolo = %{sabor: "Bolo de limão com cobertura", preco: 20, cobertura: true}
%{sabor: a } = bolo # a não é o melhor nome de variável, mas deixei aqui pra ficar claro que é uma variável
iex> a
"Bolo de limão com cobertura"
```

* O que aconteceu ai em cima?

  * Nós temos um map que nós chamamos de bolo

  * Para que o match de certo, bolo precisa ter uma chave chamada `sabor`. Caso tenha, quero salvar o valor de `sabor` em uma variável `a`

  * Além de estarmos salvando o valor em uma variável, nós também estamos verificando se a estrura que chgou para nós é a que esperamos :D

* MAS O QUE ACONTECE quando tentamos dar match em um valor que não tem naquele map????? error de pattern match (:

```elixir
%{confeiteiro: b} = bolo
** (Matcherror) no match of right hand side value ...
```

* E se a unica coisa que importa é saber se o valor é um map, independente do conteudo dele?

```elixir
iex> %{} = bolo
%{sabor: "Bolo de limão com cobertura", preco: 20, cobertura: true}
```

* Também podemos usar pattern match em mapas para checar alguns valores E extrair valores outros ao mesmo tempo

```elixir
bolo = %{sabor: "Bolo de limão com cobertura", preco: 20, cobertura: true}
%{sabor: "Bolo de limão com cobertura", valor: a } = bolo
iex> a
20 # Se o sabor tivesse outro nome, não ia rolar o match
```

* No exemplo acima nós comparamos um valor E extraimos outro. Mas se quisermos comparar E extrair o **mesmo** valor?

```elixir
bolo = %{sabor: "Bolo de limão com cobertura", preco: 20, cobertura: true}
%{sabor: a = "Bolo de limão com cobertura" } = bolo
iex> a
"Bolo de limão com cobertura" # se o sabor fosse outro, não ia dar match
```

* caso a gente não queira salvar o valor de `sabor` a variável, nós podemos sempre usar o `^`

```elixir
limão = "Bolo de limão com cobertura"
bolo = %{sabor: "Bolo de limão com cobertura", preco: 20, cobertura: true}
%{sabor: ^limão } = bolo # vai dar match, sem salvar o valor do sabor à variável "limão"
```

### MAP VS LIST

* Então enquanto listas levam vantagem sobre mapas quando queremos armazenar um número indefinido de itens, mapas levam a vantagem quando precisamos acessar facilmente seus valores. (lidar com o posicionamento disso)

* No pattern match de listas nós precisamos saber o tamanho de uma lista, ou usar um `[head|tail]` - e mesmo assim é limitado sempre ao primeiro elemento

* Já no pattern match de um map, nós não precisamos saber todos as chaves que vem em um map. Se soubermos apenas uma chave que está la dentro, conseguimos fazer pattern match.

* É importante saber que as chaves de um map não possuem ordem (Esse conhecimento sempre me vem a cabeça quando preciso fazer um assert de um teste unitário)

## Map vs Keyword Lists

* Uma keyword list é uma lista contendo tuplas com 2 elementos

```elixir
iex> [a, b, c] = [a: 1, b: 2, c: 3]
iex> a
{:a, 1}
iex> b
{:b, 2}
iex> c
{:c, 3}
```

* mas se no final Keyword list é uma forma de guardar elementos com chave e valor, porque não usamos map que faz a mesmissima coisa?

* Uma vantagem de Keyword lists em relação a maps é que nós podemos ter chaves repetidas.

* Vale lembrar que uma keyword list ainda é uma lista, então tem todas as suas limitações.

* O Uso mais comum de keyword list é quando estamos importando um módulo a partir de outro

```elixir
import String, only: [pad_leading: 2, pad_leading: 3]
# Aqui estamos importando as funções pad_leading de aridade 2 e 3 do módulo string
```

## Matching Structs

* Structs são extensões de maps, o que significa que podemos usar o que nós aprendemos em maps e aplicar em structs :D

* Structs servem para representar uma estrutura consistente, que possui o mesmo conjunto de chaves pelo código

* É impossível criar uma struct com uma chave que não existe dentro da sua estrutura

* E a graça de usar structs é que a verificação ocorre em tempo de compilação, então é uma boa idéia usar structs para ter essa camada a mais de validação.

* A forma defazer pattern match com struct é a mesma que com maps

```elixir
# Esse aqui é um struct de data: ~D[2018-01-03] (mas adianto que no dia a dia a gente vai representar a maior parte dos nossos structs de outra forma. Já chego lá)

iex> date = =~D[2018-01-03]
iex> %{year: year} = date
iex> year
2018
```

### Sigils

* Sigil são formas simplificadas de representar valores

* Reconhecemos um sigil porque eles começam com `~` e são acompanhados de letras (como `~D` para data ou `~r` para regex)

```elixir
iex> my_regex = ~r/foo|bar/
iex> "foo" = regex
true
iex> "pastel" = regex
false
###########
# Sigil de strings que nos economiza o tempo de digitar aspas duplas e virgula
iex> ~w(brigadeiro beijinho mesclado)
["brigadeiro", "beijinho", "mesclado"]
```

* A lista completa de sigils é fácil de encontrar no site oficial da linguagem

* É possível criar nossos próprios sigils, mas como isso não é comum, não vou abordar aqui

### Criando nossas proprias Structs

* É possível criar structs para identificar que tipos de dados nós temos ali

* Usamos MUITO esse tipo de dado com elementos que vem de banco de dados

* A diferença básica de uma Struct que nós criamos para um map, é que a struct possui um nome

```elixir
%User{email: "rachel@email.com", pastel_favorito: "queijo"}
```

* (Não conseguimos criar uma estrutura assim no terminal. Pra ela funcionar, precisamos criar ela dentro de um módulo e talz)

* Importante lembrar que uma struct NÃO vai dar match com um map

```elixir
iex> %Date{day: a} = %{day: 1}
** (MatchError) no match of right hand side value: %{day: 1}
```

#### Conteudo bonus que eu tirei do getting started

[(aqui, ó)](https://elixir-lang.org/getting-started/structs.html)

* Aqui um exemplo básico de como criamos uma struct

```elixir
iex> defmodule User do
...>   defstruct [:email, name: "John", age: 27]
...> end
```

* `:email`, por exemplo, é uma chave sem valor default (opcional), enquanto `:name` e `:age` possuem valores default

* É importante definir as chaves opcionais antes das chaves com valores default. Caso não for feito assim, não vai dar certo

* **IMPORTANTE**: uma struct só faz validação se as chaves estão lá. Não funciona para fazer validação de tipo dos valores

# Control Flow with functions

* Em uma aplicação nós precisamos cobrir uma variedade de cenários. Se recebemos dado x, fazemos a. Se recebemos o dado y, fazemos b

* Até então agente tem usado patterm match apenas pra verificar se um valor é igual ao outro e extrair algumas variáveis

* Mas patterm match é muito bom pra controlar fluxo da operação

* Vimos no capitulo 2 que conseguimos criar várias funções com o mesmo nome, e é juntando esse conhecimento com pattern match que conseguimos controlar fluxo da aplicação

```elixir
defmodule NumberCompare do
  def greater(number, other_number) do
    check(number >= other_number, number, other_number)
    #estamos passando uma expressão boolean e dois integers como argumentos
  end
  
  #estamos criando uma função auxiliar que checa pra nós o resultado.
  defp check(true, number, _), do: number
  defp check(false, _, other_number), do: other_number
  # A primeira função faz pattern match com a expressão booleana passada. 
  # Verifica se é true. Caso sim, retorna number
  # A outra função serve para o caso da expresão retornar false
end
```

* Explicamos acima que o caracter `_` funciona para nós como coringa. Usamos ele em ambas as funções check, porque no final das contas aqueles valores não são importante para nossa função

* **IMPORTANTE:** A ordem das funções faz TODA diferença.

* Na hora de executar o código, elixir vai primeiro verificar se tem um match na primeira função, e só passa pra próxima função se não der match

* **OUSEJA:** O caso mais genérico deve vir por ultimo. A ordem das funções de mesmo nome deve ser +- essa aqui:

  1. Verificar se os argumentos da função da match em um caso super especifico

  2. Verificar se os argumentos dão match em casos menos específicos

  3. Outros (se a função for receber algo diferente do que nós estamos esperando)

* Mudando de assunto mas ainda falando sobre esse pedaço de código, vale atentar que estamos escrevendo a função `check` usando `defp`, que significa que essa é uma função privada.

## Applying Default Values for Functions

* Conseguimos definir valores padrões para usar em uma função com o operador `\\`

```elixir
defmodule Checkout do
  def total_cost(price, quantity \\ 10), do: price * quantity
end
###################
iex> Checkout.total_cost(12, 5)
60
iex> Checkout.total_cost(12)
120
```

* Por debaixo dos panos elixir cria duas funções, uma com um argumento, e outro com 2

```elixir
defmodule Checkout do
  def total_cost(price), do: total_cost(price, 10) # *
  def total_cost(price, quantity), do: price * quantity
end

# * Repare que ele chama a si, mas passando nosso valor padrão como segundo argumento
```

* É importante apontar que só podemos usar um valor padrão por argumento por função, então nada de `def total_cost(price, quantity \\ 10)` e `def total_cost(price, quantity \\ 50)`. Não vai funcionar.

# Expanding Control with guard clauses

* Muitas vezes criar funções auxiliares para fazer checagem pode deixar o código inflado e complicado de manter

* Nós conseguimos melhorar isso com guard clauses :D

* mas qué qué isso?

* Guard Clausules nos permite usar **Expressões Boleanas** nas nossas funções

```elixir
defmodule NumeberCompare do
  def greater(number, other_number) when number >= other_number, do: number
  def greater(_, other_number), do: other_number
end
```

* (parando a explicação de guard clausules no meio só pra apontar nesse exemplo para algo que eu falei anteriormente: a ordem das funções. Sendo a mais específica a primeira, e a mais genérica a segunda)

* (okei, voltando a programação normal)

* O `when` ali no nosso código, entre os parametros e o resultado da função, é a nossa `guard clausule`

* As guard clausule funcionam como uma verificação. "Eu só vou executar esse `do` se os parametros dessa função atenderem os meus requisitos" (que nesse caso é quando o primeiro numero for maior ou igual que o segundo)

* Caso não atenda a verificação da guarde clausule, ele vai passar pra próxima função

* Conseguimos por mais de uma guard clausule por função usando `and` e `or`

```elixir
defmodule Checkout do
  def total_cost(price, tax_rate) when price >= 0 and tax_rate >= 0 do
    price * (tax_rate + 1)
  end
end
```

* Vamos abrir outro parenteses aqui.

  * Vamos supor que passamos um tax_rate não numérico (algo como "pastel").

  * O retorno disso vai ser um erro, mas não o erro que nós estamos esperando.

  * Uma string vai passar pela guard clausule, porque em elixir **É POSSÍVEL COMPARAR ELEMENTOS DE TIPOS DIFERENTES**

  * Isso é bastante prático quando queremos organizar listas com itens diversos.

  * Elixir é uma linguagem dinamicamente tipada, então não precisamos ficar tão na defensiva sobre tipos

  * Se precisarmos fazer checagem de tipos o elxir tem uma função útil pra isso, que é o `Kernel.is_integer/1` (lembrando aqui que não há necessidade para chamar o módulo `Kernel` nas funções de Kernel. Basta chamar um `is_integer`)

### Guard clausules em funções anônimas

* Também é possível usar guard-clausules em funções anônimas

```elixir
number_compare = fn
  number, other_number when number >= other_number -> number
  _, other_number -> other_number
```

### Guard Clausules e funções

* Não podemos usar funções comuns em guard clausules

* Porque o algoritmo que checa se dar match precisa ser muito rápido, então não há tempo pra compilar funções E fazer o match

* O que significa que só funções puras são possíveis

* Existem algumas funções macro que são permitidas usar em guard clausules. Se você olhar a [documentação oficial](https://hexdocs.pm/elixir/Integer.html#guards), na parte de strings tem uma seção chamada "Guards". Lá estão listadas as funções que são permitidas se usar como guard Clausule (mais especificamente `is_odd` e `is_even`)

* Porém, pra que isso aconteça, precisamos importar o módulo `Integer`, por exemplo

```elixir
defmodule EvenOrOdd do
  require Integer #importante!
  
  def check(number) when Integer.is_odd(number), do: "odd"
  def check(number) when Integer.is_even(number), do: "even"
end
```

* Vale a obs que o `require` é lexicamente escopado. então se ele for chamado dentro de um modulo, ele só vai funcionar praquele módulo. se for chamado dentro de uma função, só funcionar dentro daquela função

### Criando nossas próprias funções macro

```elixir
defmodule Checkout do
  defguard is_rate(value) when is_float(value) and value >= 0 and value < = 1
  defguard is_cents(value) when is_integer(value) and value >= 0
  # Aqui em cima são as funções macro que criamos :D 
  
  # e aqui a gente usa elas :D 
  def total_cost(price, tax_rate) when is_cents(price) and is_tax(tax) do
    price * tax
  end
end
```

* Não são todas as funções podem ser usadas para se usar em guard clausules, então ficar esperto

# Elixir Controw-flow Structures

* Elixir tem pattern match e, geralmente, é nosso principal controlador de fluxo, mas isso não significa que não temos outras formas de controlar o fluxo. Ifs ainda são bastante úteis, e também temos case e cond

## Case: Control with pattern matching

### O que é

* Cases são úteis quando queremos verificar uma expressão que pode ter vários pattern matchs

### quando é útil?

* Quando temos uma função que pode ter efeitos inesperados

### Exemplo

```elixir
user_input = IO.gets "Write your ability score: \n"
result = case Integer.parse(user_input) do
  :error -> "invalid score"
  {ability_score, _} -> 
    modifier = (ability_score - 10) / 2
    "Your ability score é #{modifier}"
end

IO.puts result
```

* No exemplo acima nós pedimos pro usuário digitar um numero. Se não fosse um numero, ele ia cair na primeira condição do nosso case (`:error`). Se fosse um numero, ele printa qual que é o score

* Todas as linhas entre o `do` e o `end` podem ser usadas para gerar clausulas

* do lado esquerdo do `->` é o pattern match que queremos comparar, e do lado direito é a "ação" que queremos que ocora em caso de match (e isso pode ser uma linha ou multiplas linhas)

* É uma boa pratica usar o retorno do case para dar continuidade nas nossas funções. (como assim? seguem exemplos)

```elixir
### BOM EXEMPLO
user_input = IO.gets "Write your ability score: \n"
result = case Integer.parse(user_input) do
  :error -> "invalid score"
  {ability_score, _} -> 
    modifier = (ability_score - 10) / 2
    "Your ability score é #{modifier}"
end

IO.puts result

### EXEMPLO NÃO MUITO BOM
user_input = IO.gets "Write your ability score: \n"
case Integer.parse(user_input) do # aqui não foi atribuido a uma variavel
  :error -> IO.puts "invalid score" #a ação ocorre aqui, o que não é legal
  {ability_score, _} -> 
    modifier = (ability_score - 10) / 2
    IO.puts "Your ability score é #{modifier}" # aqui também 
end
```

## Cond: Control with Logical Expressions

### O que é

* Usamos quando queremos checar diferentes variáveis em expressões lógicas

### No que é útil

* É útil quando não precisamos de pattern match pra resolver um problema

### Exemplo

```elixir
{age, _} = Integer.parse IO.gets("Person's age:\n")
result = cond do
  age < 13 -> "kid"
  age <= 18 -> "teen"
  age > 18 -> "adult"
end
```

* A estrutura é bem similar ao case, mas do lado esquerdo nós queremos lidar com expressões lógicas

* Quando a expressão resulta em algo `truthy`, aquele resultado vai dar bom. Caso não, ele passa pro próximo

* (Lembrando que em elixir truthy é qualquer valor diferente de `nil` ou `false`)

## Taking a look at our old friend if

* Não vou explicar muito como o if funciona, pois é similar em tudo que é linguagem da vida

* Mas segue a sintaxe

```elixir
defmodule NumberCompareWithIf do
  def greater(number,other_number) do
    if number >= other_number do
      number
    else
      other number
    end
  end
end
```

* Similar ao `if` também temos `unless`. Enquanto o if vai dar bom se o valor for `truthy`, o unless vai dar bom se o valor for `falsy`

  * mas Costumamos a evitar o `unless` pois deixa o código mais confuso. É boa prática sempre deixar o if bem explicito e evitar negativas

* é possível escrever ifs de uma linha `if(number >= other_number, do: number, else: other_number)`

## Uma breve conclusão aqui

* User muito control-flow vai deixar seu código mais imperativo

* Por outro lado usar muitas guard clausules e pattern match as vezes deixa seu código com uma menor legibilidade.

* O ideal é tentar balancear ambos os casos. Não deixamos de usar ifs nos nossos códigos, mas geralmente seu código fica mais fácil de entender com pattern match.

* Escolher quando é melhor usar um ou outro vai ficar mais fácil na prática.  O ideal é sempre optar pela opção mais legível.
