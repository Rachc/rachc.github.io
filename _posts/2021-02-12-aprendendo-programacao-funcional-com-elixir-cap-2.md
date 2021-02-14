---
layout: post
title:  Aprendendo programação funcional com elixir - cap2
date:   2021-02-12 19:30:00 -0300
image:  functional01.jpg
tags:   Funcional, Elixir
---
# Intro
Pra quem chegou agora eu, junto com a Elaine Watanabe e a Juliana Helena, estamos estudando o livro Learning Functional Programming with elixir do autor Ulisses Almeida.

Estamos fazendo lives [aqui](https://www.youtube.com/channel/UCHDwILYk5-LpMAUIUcPSZMw), e essas são as minhas anotações (:

Pra quem perdeu a primeira parte, [aqui está o link da live](https://www.youtube.com/watch?v=DYszf5MF8fA), e [aqui as anotações](({{ site.baseurl }}/2021/02/09/aprendendo-programacao-funcional-com-elixir-cap-1/))
e o link da live sobre o capítulo 2, referente às essas notas [está aqui](https://youtu.be/wmLJBtjGJgk)

# Trabalhando com variáveis e funções
## Representando valores (tipos)
* Valores são tudo aquilo que pode representar dados. Pode ser a quantidade de transações de um usuário, o preço de um produto ou qualquer coisa que o um programa possa receber de input, computar e gerar resultado

* Se digitarmos `10` no nosso terminal, ele retorna 10 e isso é um valor, que representa um número e possui tipo `integer`
* Digitando `"Elixirlab é top"` nós temos um outro tipo de valor, dessa vez do tipo `String`.

* A tabela abaixo tem alguns tipos que podemos encontrar em elixir. A lista completa [pode ser encontrada na doc oficial](https://hexdocs.pm/elixir/typespecs.html#basic-types) - em inglês.

| Tipo  |            Útil para          |                Exemplos             |
|------ |-------------------------------|-------------------------------------|
|string |Textos                         |"Hello World", "ElixirLab"           |
|integer|números inteiros               |42, 33, 583, 0301, -23               |
|float  |números reais                  |42.8, 3.1415, -3.2                   |
|boolean|operadores lógicos             |true, false                          |
|atom   |identificadores                |:ok, :error, :pastel                 |
|tupla  |coleções de valores definidos  |{:ok, "top}, {123,456}               |
|list   |coleções de valores indefinidos|[3,1], ["A", "b"]                    |
|map    |mapa de chave e valor          |%{id:1, name: "Rach"}, %{3 => pastel}|
|nil    |ausência de valor              |nil                                  |

* Um pequeno parênteses sobre atom: Eles são uma constante, e a melhor analogia pra entender melhor é que eles funcionam como uma etiqueta. Se você identificar um pote de tempero com uma etiqueta "orégano" você está informando que dentro daquele pote tem orégano (a etiqueta seria um atom)

* Os valores de boolean e nil são considerados atoms (`:nil == nil`, `:true == true`, `:false == false` )

* (Rachel: em elixir a gente não pode gerar tipos novos, como acontece em OO. No máximo você cria umas structs, que é um map mais avançado, que será visto mais pra frente)

## Executando código e gerando um resultado (operadores)

* Quando usamos o terminal (iex), digitamos e apertamos enter, o computador vai processar aquele valor.

* Se você digitar `42`, ele vai retornar `42` da mesma forma que se você digitar `1+1` ele retorna `2`

```elixir
iex> 42
42
iex> 1+1
2
iex> (2+2) * 3 # <---- vai respeitar as regras matemáticas
12
iex> 2 + 2 * 3
8
```

* Geralmente quando você tenta criar expressões com tipos não compatíveis, ele retorna um erro

```bash
iex> "olar" + 33
** (ArithmeticError) bad argument in arithmetic expression
    :erlang.+("olar", 33)
```

* Mas não podemos pensar que usar operadores em tipos diferentes pode sempre dar ruim. Quando juntamos  `float` com `integer`, funciona

```elixir
iex> 12 + 8.5
20.5
```

* O exemplo acima porque um seja `integer` e o outro seja `float`, o elixir ver ambos como `number`, que é a junção de ambos os tipos

### Operadores (adicionado por Rachel)

* A maioria dos operadores que conhecemos de outras linguagens funciona também em elixir

* `+`, `-`, `/`, `*` para fazer operações aritméticas entre números, de adição subtração, divisão e multiplicação respectivamente

* `==`, `!=`, `<`, `<=`, `>`, `>=` para comparar dois valores e retornar um boolean. Respectivamente ele compara se os valores são iguais, diferentes, menor, menor igual, maior e maior igual (`:nil == nil`, por exemplo (vai retornar true, inclusive))

* Elixir tem dois operadores que não são tão comuns em outras linguagens, que é o `++` e o `<>` que servem para concatenar

* O `++` serve para concatenar listas

```elixir
iex> [1,2] ++ [3,4]
[1,2,3,4]
```

* Enquanto o `<>` junta strings

```elixir
iex>"Sorvete " <> "de " <> "Creme."
"Sorvete de Creme."
```

* Vale frisar aqui que o operador `+` não funciona para juntar strings, apenas o `<>`

```elixir
iex> "Sorvete " + "de " + "Creme."
** (ArithmeticError) bad argument in arithmetic expression: "Sorvete " + "de "
    :erlang.+("Sorvete ", "de ")
```

### Criando expressões lógicas

* Elixir também tem operadores para comparar se duas ou mais expressões são verdadeiras ou falsas

* `and` e `&&` retorna true se as duas expressões forem verdadeiras. Se uma for falsa, ele retorna falso `1+1 == 2 && 2 + 2 == 4` vai retornar `true`, por exemplo, enquanto `1 + 1 == 3 && 1 + 1 == 2` vai retornar `false` (o `&&` pode ser substituído por `and`)

* `or` e `||` retorna true se pelo menos uma das expressões retornar verdadeiro.

* `not` e `!` vai retornar true se o resultado for o oposto do que nós esperamos (confuso, mas pensa que `!true` é `false` )

#### Quando usar um e quando usar outro
* `and` e `or` só funcionam com expressões booleanas e retornam sempre booleanos

* O `&&` e `||` funcionam com expressões truthy e falsy e o retorno pode variar

* Quando usamos `&&` e `||` o regra de retorno é o seguinte:

* Quando falamos de `&&`

```elixir
iex> is_integer(2) && 3 #o resultado da expressão é true. Ele vai retornar o ultimo valor
3
iex> 3 && is_integer(2) #o resultado da expressão é true. Ele vai retornar o ultimo valor
true
iex> 3 && is_nil(2) #o resultado da expressão é false
false
iex> is_nil(2) && 3 #o resultado da expressão é false
false
```

* Quando falamos de `||`

```elixir
iex> is_nil(2) || 3 # um é falso, outro verdadeiro, vai retornar o valor verdadeiro
3
iex> 3 || is_nil(2)# um é falso, outro verdadeiro, vai retornar o valor verdadeiro
3
iex> is_integer(2) || 3 # vai retornar o valor da expressão, que é true
true
iex> 3 && is_integer(2) # vai retornar o valor da expressão, que é true
true

```

#### Truthy e Falsy (adicionado por Rachel)

* O Elixir também consegue comparar valores `truthy` e `falsy`, que na real são expressões não-booleanas

* Em elixir os únicos valores `falsy` são `false` e `nil`, de resto, qualquer valor é considerado true (inclusive `[]` ou `""` ou `0`)

* É como se a gente estivesse comparando a existência de alguma coisa (valores `truthy` com a ausência `falsy`)

* E por que isso é importante? Imagina que você está buscando um usuário no banco. Se ele existir, o elixir considera isso um valor `truthy` e se ele não existir, o valor é `falsy`


## Associando valores a variáveis

* A definição de uma variável é bem parecida com o que acontece nas outras linguagens.

* `sentido_da_vida = 42` <- você define um nome na esquerda, acrescenta um `=` e o valor na direita

* O Ulisses compara a criação de variável como colocar uma etiqueta em uma caixa (que você coloca o nome na direita e aponta o valor dessa etiqueta na esquerda)
* Também podemos usar variáveis para compor o valor de outras variáveis:

```elixir
x = 3
y = 7
z = x + y # 10
```

* Apesar do elixir não se importar com o nome da variável e funcionar direitinho, devemos lembrar que um código é feito para as pessoas que vão dar manutenção a ele, então colocar nomes que fazem sentido é o mais indicado.

* Ao invés de x, y e z podemos usar nomes com mais significado como:

```elixir
z = x + y #ruim
salarios_totais = salario_ana + salario_vera #bom
dano_final = ataque + modificador #bom também
```

#### Convenções de nome da comunidade

* Geralmente usamos `snake_case`, o que significa que as variáveis são escritas em letra minúscula e separadas por underline (`_`)

* Em elixir não é permitido começar variáveis com letras maiúsculas, pois isso são reservado para módulos (Vamos ver módulos depois)

```elixir
arvores = 5000 # bom
Arvores = 5000 # não vai compilar
total_de_arvores = 5000 #bom
totalDeArvores = 5000 # vai funcionar, mas não segue o guia de estilos de elixir
```

## Criando funções anônimas

* Podemos pensar em funções anônimas como subprogramas dos nossos programas

* Elas servem para facilitar algumas tarefas repetitivas

```elixir
iex> "Pastel de carne"
iex> "Pastel de queijo"
iex> "Pastel de frango"
```

* No lugar de digitar sabores de pastel repetidamente, podemos criar uma função pra isso

* O primeiro passo é abstrair o que se repete e o que muda. O que muda pode ser transformado em uma variável

```elixir
iex> sabor = "carne"
iex> "Pastel de " <> sabor
"Pastel de carne"
```

* O próximo passo é transformar em uma função usando a variável `sabor` em um parâmetro para essa função

* A sintaxe para criar uma função anônima é a seguinte:

* `nome_da_variavel = fn parametro1, parametro2 -> corpo_da_função end`

* e para usar uma função anonima que atribuimos a uma variavel

* `variavel_que_usamos.(parametro1, parametro2)`

```elixir
iex> sabor_de_pastel = fn sabor -> "Pastel de " <> sabor end
iex> sabor_de_pastel.("queijo")
"Pastel de queijo"
iex> sabor_de_pastel.("camarão")
"Pastel de camarão"
iex> sabor_de_pastel.("chocolate")
"Pastel de chocolate"
```

* no lugar de usar o operador `<>` como concatenadores de strings, podemos usar a sintaxe de interpolação

* `sabor_de_pastel = fn sabor -> "Pastel de #{sabor}" end`

* (e isso não funciona só pra strings, mas tudo que estiver dentro de `#{}` é considerado código)

* então podemos usar `"fazendo #{1+1} teste"`, ou 

```elixir

iex> sabor_favorito = "queijo"
iex> "Garçom, me vê um #{sabor_de_pastel.(sabor_favorito)}"`
"Garçom, me vê um Pastel de queijo"`
```

* Podemos usar mais de uma linha em uma função anônima

```elixir
pedindo_pastel = fn sabor ->
  sabor_favorito = "Pastel de #{sabor}"
  "Garçom, me vê um #{sabor_favorito}"
end
```

* e também é possível usar funções sem argumentos

```elixir
iex> um_mais_um = fn -> 1 + 1 end
iex> um_mais_um.()
2
```

### Funções como cidadãos de primeira classe

* Quando falamos que uma função é um cidadão de primeira classe **não** queremos dizer que ele é mais especial do que outros valores

* Quer dizer que ele é tratado de forma igual a qualquer outro valor.

* Da mesma forma que temos o tipo `String` ou `integer`, também temos o tipo `function`

* O que significa que podemos passar uma função como argumento da outra

```elixir
iex> cumprimento = fn nome, saudacao -> saudacao.(nome) end
iex> saudacao_oi = fn nome -> "oi, #{nome}" end
iex> saudacao_olar = fn nome -> "olar, #{nome}" end

iex> cumprimento("Rachel", saudacao_oi)
"oi, Rachel"
iex> cumprimento("Rachel", saudacao_olar)
"olar, Rachel"
```

* Trazendo um exemplo mais prático e mais próximo do dia a dia:

```elixir
iex> sum_2 = fn number -> number + 2 end
iex> Enum.map([1,2,3], sum_2)
[3,4,5]
```
### Compartilhando valores sem usar argumentos

* Uma função anônima com acesso às variáveis do seu entorno é uma closure. Como assim?

```elixir
iex> risoto = "funghi"
iex> risitoinho_top = fn superlativo -> "Acho o risoto de #{risoto} #{superlativo}" end
iex> risotinho_top.("top demais")
"Acho o risoto de funghi top demais"
```

* No exemplo acima temos uma função anônima que está usando uma variável em um escopo externo, mas que a função tem contexto

* (Achei essa definição de closure em um [post da Charlotte](https://imasters.com.br/desenvolvimento/closures-funcoes-anonimas-e-funcoes-nomeadas) no imasters)

#### Escopo vs escopo léxico

* Escopo é parte de um programa

* E o escopo léxico tem relação com a visibilidade das variáveis do código.

* Uma variável criada no corpo de um módulo vai ser acessível para todo aquele módulo

* Mas uma variável criada dentro de uma função só vai ser acessível dentro daquela função

* (E a função vai ter acesso não só as variáveis criadas dentro dela, mas também as que foram criadas fora dela)

* No exemplo acima a variável `superlativo` só existe dentro da função anônima `risotinho top`, enquanto a variável `risoto` pode ser acessada em qualquer um dos escopos.

#### Sobre precedências de variáveis em diferentes escopos

* Se existe uma variável de mesmo nome, sendo uma dentro de uma função e outra fora, na função a precedência maior é da variável criada dentro da função:

```elixir
iex> chiclete = "morango"
iex> babaloo = fn chiclete -> "o melhor chiclete é #{chiclete}" end
iex> babaloo.("tuti-fruti")
"O melhor chiclete é tuti-fruti"
```

## Nomeando funções

* Aprendemos a criar funções anônimas e achamos elas maravilhosas, mas em um codebase grande não é viável lidar com isso

* Elixir tem suas funções nomeadas embutidas, mas nós também podemos criar.

* Funções nomeadas são criadas dentro de módulos

* (Rachel aqui: é o mais próximo de um objeto que nós vamos ter. Módulos são como uma caixinha de ferramentas e cada função é uma ferramenta diferente, a diferença é não teremos estado)

* Podemos usar aliases ou atoms para nomear um módulo

* Em elixir, tudo que começa com letra maiúscula é um alias, e todo alias é transformado em atom no tempo de compilação

* (Achei isso confuso, já que eu entendo alias como uma forma de você "simplificar" o nome de um modulo. ex: `alias MeuApp.Usuarios.Usuarios` me permite chamar todo esse modulo apenas por `Usuarios`)

* Ainda sobre aliases virarem atoms:

```elixir
iex> String == :"Elixir.String"
true
```

* OKEI

* Voltando a idéia que um módulo é uma caixinha de ferramentas, e uma função é uma ferramenta.

* Para usarmos uma função nomeada nós usamos a sintaxe `NomeDoModulo.funcao(parametros)`

* `String.upcase("uhuuuu")` <- String é o módulo e upcase é a função

* Em elixir também podemos omitir parênteses, para todas as funções (exceto pipe), mas é dsaconselhavel usar por motivos de legibilidade

```elixir
iex> IO.puts "top demais" #ok, ainda é legível
iex> Enum.map ["a", "b", "c"], &String.upcase/1 # desaconselho fortemente
iex> IO.inspect "a", label: :my_a, limit: :infinity # olhaissoai! é só confuso!
```

### Funções nomeadas em elixir

* O elixir vem com módulos e funções embutidos dentro dele. 

* O que eu, Rachel, mais uso no dia a dia são as funções do módulo `Enum` 

* mas também temos os módulos `String`, `Integer`, `Float`, `IO`, `Kernel`, `Map`, `List`

* (Um parênteses aqui. As funções de `Kernel` podem ser chamadas sem o nome do módulo. `Kernel.is_number("olar")` pode ser chamada só como `is_number("olar")`)

* Dica de Rachel para saber as funções dos módulos nativos do elixir:

  * Abrir o terminal e ir no IEX

  * Digitar o nome do módulo (`Enum`, por ex)

  * Apertar tab

  * SE VOCÊ QUISER SABER COMO UMA FUNÇÃO FUNCIONA:

    * `h Enum.map`

    * Sério, a ajuda do elixir embutida no terminal é uma coisa linda demais! tem uma extensa explicação e bons exemplos <3 e dica: dá pra fazer esse nível de ajuda na sua aplicação usando docs, mas explicaremos isso depois (acho)

### Criando módulos e funções
* Nós podemos colocar um módulo em qualquer lugar do projeto que ele vai ser acessível pra todo mundo (geralmente na pasta lib, mas não encanaremos isso por hora)

* A extensão de um módulo é `.ex`

* Sua sintaxe é:

```elixir
defmodule NomeDoModulo do
end
```

* Quando quisermos criar uma função desse módulo usamos a sintaxe

```elixir
defmodule NomeDoModulo do
  def funcao_marota(parametro) do
    # Corpo da função
  end
end
```

* (Já já falamos sobre como usar um módulo. aguenta ai!)

* Mas a real é que dentro de um módulo podemos criar o que quisermos. Variaveis, funções anônimas, funções nomeadas (públicas ou privadas) e por aí vai

* Só relembrando que as convenções de nome para módulos é `CamelCase` e para funções é `snake_case`

* Se quisermos acessar essa função de outra parte do código usamos `NomeDoModulo.funcao_marota(valor)` 

* Mas podemos usar também o módulo no terminal:

  * **No terminal, vamos para a pasta que está nosso módulo** <- importante!!!!!
  
  * iex

  * `c("nome_do_arquivo.ex")` (`c` e de compile e load, [segundo a doc oficial](http://erlang.org/documentation/doc-5.3/doc/getting_started/getting_started.html))

  * `NomeDoModulo.funcao_marota(valor)`

* Existem duas formas de você criar uma função nomeada dentro de um módulo. A primeira é como vimos, com multiline, mas também podemos criar a mesma função em uma linha só

* `def funcao_marota(valor), do: IO.inspect(valor)`

* A escolha de quando é melhor uma ou outra vai do gosto do freguês

* É boa prática nomear o seu módulo de acordo com a estrutura de pastas que ele está inserido

* Então se sua estrutura de pastas é algo como `projeto > lib > comidas > massas > macarrao.ex ` é interessante chamar seu módulo de `Comidas.Massas.Macarrao`

* A vantagem disso é que cada módulo tem que ter um nome único, então se você usar o padrão, consegue não só diminuir a chance de ter módulos de nomes repetidos, mas também ajuda a organizar melhor o seu projeto

### Importando funções nomeadas

* Ocasionalmente queremos usar as funções em outros módulos, e isso é fácil de fazer com a sintaxe `NomeDoModulo.funcao()`, mas as vezes faz sentido que você trate essa função como as funções tipo `Kernel`, que é possível chamar só `funcao()`.

* Dá pra fazer isso usando import

* No exemplo abaixo usamos as funções `write` e `read` do módulo `File`

```elixir
defmodule TaskList do
  @file_name "task_list.md"
  
  def add(task_name) do
    task = "[ ] " <> task_name <> "\n"
    File.write(@file_name, task, [:append])
  end
  
  def show_list do
    File.read(@file_name)
  end
end
```

* podemos simplificar esse módulo se importarmos as funções `read` e `write` do módulo `File`

```elixir
defmodule TaskList do
  import File, only: [write: 3, read: 1]
  
  @file_name "task_list.md"
  
  def add(task_name) do
    task = "[ ] " <> task_name <> "\n"
    write(@file_name, task, [:append])
  end
  
  def show_list do
    read(@file_name)
  end
end
```

* Embora mais prático, usar import pode deixar o código confuso. No exemplo acima nós temos um módulo pequeno, mas imagina que temos um módulo maior, com muitas funções. Em algum momento não vamos saber com clareza a origem das funções `read` e `write`.

* Imagina que você não apenas tem um módulo grande, como importa várias funções. O seu processo de debug vai ser mais tenso.

* Então é bacana usar import com parcimônia

#### Aridade (Adicionado por Rachel)

* mas o que vem a ser esses números depois das funções que queremos importar?

* Chamamos isso de **Aridade**. 

* Aridade é o número de argumentos que uma expressão precisa. Nós vamos esbarrar várias vezes com a sintaxe `File.read/1` ou `File.write/3`

* E porque isso é importante?

* Em elixir podemos ter incontáveis funções com o mesmo nome, e o elixir entende que funções de mesmo nome com aridades distintas são funções distintas. 

* Se você for no terminal, dentro do seu iex, e digitar `h Enum.all?` e apertar tab, você vai ver que temos `Enum.all?/1` e `Enum.all?/2` 

### Usando funções nomeadas como valores

* Podemos capturar funções anônimas em variáveis `sum_2 = fn number -> number + 2 end`

* mas conseguimos capturar funções nomeadas em uma variável?

* `first = Sring.first` vai retornar um erro (mesmo se você tentar especificar a aridade `first = String.first/1`). Elixir vai dar uma embananada

* mas e se eu quiser fazer isso? tem jeito?

* Sim! podemos embrulhar a função nomeada em uma função anônima

```elixir
iex> first = fn string -> String.first(string) end
iex> first.("Olar")
"O"
```

#### Operador &

* O operador `&` é uma forma de simplificar a escrita de funções anônimas

* (Rachel: Particularmente eu acho que confunde um pouco quem é iniciante na linguagem. Não é a sintaxe mais legível do mundo, mas depois que você se acostuma, ela simplifica um pouco a vida. Então o conselho é usar com parcimônia)

* o operador `&` captura uma função anônima.

* Então no lugar de escrever `first = fn string -> String.first(string) end` podemos escrever `first = &String.first/1`

* Como o operador `&` captura uma função anônima, você também pode usar para escrever funções anônimas

```elixir
iex> custo_total = &(&1 * &2)
iex> custo_total.(10,2)
20
```

* No exemplo acima nós usamos `&1 e &2` para lidar com os argumentos que a função iria receber. Ela poderia muito bem ter sido escrita dessa forma:

```elixir
iex> custo_total = fun number1, number2 -> number1 * number2 end
iex> custo_total.(10,2)
20
```

* também é possível não usar os parenteses, mas eu desaconselho fortemente, já que é fácil ficar confuso demais `multiplica_por_2 = & &1 * 2`

* **Como eu entendi o operador &**: entendi que eu consigo trocar o combo `fn -> end` por `&` na maioria das vezes (não consigo fazer a troca se a função não tem argumento. `fn -> 2 end` não pode ser trocado pelo operador `&`, por exemplo)

* No final das contas o operador `&` deve ser usado com precaução. Ocasionalmente ele vai te ajudar a transformar um código verboso em algo simples, ocasionalmente ele vai trazer uma ilegibilidade desnecessária. 

* (mas de qualquer forma, eu treinaria um pouco com ele, pois ele é comumente usado)
