---
layout: post
title:  Aprendendo programação funcional com elixir - cap6
date:   2021-05-13 08:00:00 -0300
image:  functional01.jpg
tags:   Funcional, Elixir
---
# Disclaimer
* Os posts dessa série são minhas anotações pessoais dos meus estudos do livro "Aprendendo programação funcional com elixir.

* Eles não passaram por revisão, possuem typos e alguns erros. 

* Estão mais próximos de pensamentos desconexos do que de um texto estruturado.

* Dito isso, segue anotações (: 

# Mix

* Mix é uma ferramenta de CLI (comand line interface)

* Você consegue criar seu mix pra fazer meio que o que você quiser (dar inicio a uma aplicação, por exemplo).

* Alguns comandos do mix já vem por padrão (Como o mix test, por exemplo)

* A gente usa muito o "mix phx.server", ou um mix ecto.migrate

* (criar um projeto aqui e agora)
* (mostrar em que pasta vai as mix tasks que a gente cria lib > mix > taks > minha_task.ex)

* Precisa ter um use "Mix.Task" e também precisa ter a função "run"

```elixir
defmodule Mix.Tasks.Start do
  use Mix.Task
  
  def run(_), do: IO.inspect("Hello, world")
end
```

* para executar vamos no terminal, na pasta do projeto, e digitamos "mix.start"

* lembrando mais uma vez que é mix task é uma ferramenta de CLI, então se você precisa executar alguma coisa via terminal, mix.task é seu melhor amigo :D

* (é uma boa hora pra explicar como definimos o nome do modulo?)

# Struct

* Em POO um objeto é um conjunto de propriedades e métodos. Em elixir, podemos agrupar um conjunto de propriedades como um struct (O que é bastante útil quando falamos de banco de dados, mas não é só pra isso que struct servem)
* Quando temos um agrupamento de dados consistentes  na nossa aplicação, podemos criar structs (Inclusive, eles podem ser considerados um tipo novo)

* (Pra quem não lembra structs são meioq ue maps, porém mais estruturados)

```elixir
defmodule Mercadinho.Padaria.Bolo do
  defstruct sabor: nil,
            preco: 0, 
            cobertura: false,
            pronto: false 
end
```

* Podemos criar um novo bolo no terminal

```bash
iex -s mix 
####
iex> limao = %MyApp.Bolo{sabor: "bolo de limão com cobertura", preco: 30, cobertura: true, pronto: true}
iex> limao.sabor
"bolo de limão com cobertura"
```

* O nome do modulo tem ligação com a forma que os arquivos estão organizados.

* O "MyApp é o namespace". Pensa no namespace como o dominio da aplicação.

* Se nossa aplicação fosse um mercadinho o dominio poderiam ser seções do mercadinho (e bolo provavelmente estaria em padaria. Da mesma forma que morango estaria no dominio de hortifruti)

* `defmodule Mercadinho.Padaria.Bolo` (ou algo assim)

## Alias (não ta no capitulo, mas acho bem util por aqui)

* Quer dizer que toda vez que formos usar a scruct de bolo precisariamos digitar `%Mercadinho.Padaria.Bolo{sabor: "bolo de limão com cobertura", preco: 30, cobertura: true, pronto: true}`?

* Não necessariamente.  Podemos usar o `alias`, que da um apelido pra um módulo

* Usar `alias Mercadinho.Padaria.Bolo` nos permite usar apenas `%Bolo{sabor: "bolo de limão com cobertura", preco: 30, cobertura: true, pronto: true}` dentro do escopo que o alias foi chamado

```elixir
defmodule Mercadinho.Algo do
  alias Mercadinho.Padaria.Bolo
  
  #(...)
  %Bolo{sabor: "bolo de limão com cobertura", preco: 30, cobertura: true, pronto: true}
  #(...)
  
end
#(fora do escopo o alias não funciona. Aqui, por ex, %Bolo{...} não funcionaria)
```

* podemos dar outro nome pro alias

```elixir
defmodule Mercadinho.Algo do
  alias Mercadinho.Padaria.Bolo, as: Pastel
  
  #(...)
  %Pastel{sabor: "bolo de limão com cobertura", preco: 30, cobertura: true, pronto: true}
  #(...)
  
end
```

# Protocols

* Protocolos são formas de atingir o polimorfismo em elixir (que é basicamente quando queremos que um comportamento varie de acordo com o tipo de dados)

* Vamos supor que queremos saber o tipo de um dado. Podemos fazer umas várias funções que fazem a verificação

```elixir
defmodule Utility do
  def type(value) when is_binary(value), do: "string"
  def type(value) when is_integer(value), do: "integer"
  # ... other implementations ...
end
```

* Porém essa forma é complicada de se extender. Vamos supor que esse código está em uma lib, e alguém coloca na sua aplicação e cria outros tipos de dados. Como podemos extender isso ai?

* A resposta é Protocols \o/

```elixir
defprotocol Utility do
  @spec type(t) :: String.t() #vamos ver isso já já
  def type(value)
end

defimpl Utility, for: BitString do
  def type(_value), do: "string"
end

defimpl Utility, for: Integer do
  def type(_value), do: "integer"
end
```

* No exemplo acima se formos no iex e digitarmos `Utility.type(10)` ele vai retornar uma coisa, se digitarmos `Utility.type("Pastel")` ele retornará outra

* Okei, achei top, mas como eu organizo isso ai? Eu coloco todos os `defimpl` no mesmo arquivo do `defprotocol`? Ou vale a pena colocer no modulo que criamos os tipos? tem algumas regrinhas

  1. Você que criou a struct? (ex: a implementação da função vai acontecer dentro do `Mercadinho.Padaria.Bolos`? ou a implementação da função é de um tipo que está além do seu alcance? tipo `Integer`?) Se a resposta for "O struct é criação minha" -> Coloque dentro do struct. Se não, próxima pergunta:
  2. Eu que estou criando o protocolo? -> Coloque a implementação dentro do arquivo que você criou o protocolo
  3. Mas eu não sou dono de nenhum dos dois /o\ ( ~~eu tenho apenas 6 anos~~). Eu simplesmente peguei a implementação de uma lib ai e to aplicando em umas structs que não fui eu que criei -> Crie um arquivo com o nome do protocolo e coloque sua implementação lá

# Behaviours

* Behaviour é uma forma de especificar um contrato entre um módulo e o client que ta usando esse módulo. Como assim Rachel? Qué qué é isso?

* Acho importante primeiro explicar rapidão o que a gente chama de interface:

* Interface são todas as funções públicas de um módulo. Pensa que cada modulo tem um conjunto de funções, e se eu quiser lidar com alguma coisa dentro daquele módulo, eu estarei usando as funções públicas daquele módulo (`def`, e não `defp`).

* Agora vamos dizer que você ta criando uma API (ou criando uma aplicação que consome de uma API) e você precisa garantir que aquela interface pública vai ser daquele jeito. A melhor forma defazer isso é criar um contrato, e em elixir a gente faz isso com `behaviour`

* Um behaviour funciona com 2 partes: Primeiro você define, e depois você especifica onde vai implementar  (vai ficar melhor com exemplos)

* No exemplo vamos implementar um parser.
  1. Definição: Eu preciso em algum lugar explicar quais as funções que eu PRECISO ter no meu contrato. Aquelas que não podem faltar de jeito nenhum (no exemplo abaixo estamos criando um parser)

```elixir
defmodule Parser do
  @doc """ # Bom e velha doc
  Parses a string.
  """
  @callback parse(String.t) :: {:ok, term} | {:error, String.t} # Aqui que a magia acontece. Mas o que são esses :: rachel? e esse String.t? Explico já

  @doc """
  Lists all supported file extensions.
  """
  @callback extensions() :: [String.t]
end
```

* No exemplo acima criamos o contrato com as funções que precisa ter em qualquer parser que eu vou criar.

* Ok, contrato criado, e ai? qué que eu faço com isso?

* Usamos `@behaviour`

```elixir
defmodule JSONParser do
  @behaviour Parser #Aqui ta a mágica :D 

  @impl Parser
  def parse(str), do: {:ok, "some json " <> str} # ... parse JSON

  @impl Parser
  def extensions, do: ["json"]
end
```

* Outro paser :D

```elixir
defmodule YAMLParser do
  @behaviour Parser

  @impl Parser
  def parse(str), do: {:ok, "some yaml " <> str} # ... parse YAML

  @impl Parser
  def extensions, do: ["yml"]
end
```

* mas o que acontece se eu não quiser implementar uma das funções do contrato? -> Erro de compilação!

* E eu posso colocar outras funções além das que eu especifiquei no contrato? SIIIIIIIM :D

# Typespecs

* Okei, lembra que eu falei que ia explicar o que era o `::`?  Chegou a hora (quase, mas chegou)

* Elixir é uma linguagem dinamicamente tipada. O que significa que nós não precisamos nos preocupar com o tipo de nada :D

* mas se a gente quiser? Se eu precisar garantir o tipo das coisas? Comofas?

* Podemos definir o tipo dos argumentos que a gente quer que as funções tenham, bem como o seu retorno.

* Para o exemplo, vamos supor que uma função receba dois numeros, some ambos e retorne uma string "O resultado é x", sendo x a soma.

```elixir
def soma(a, b), do: "O resultado é {a+b}"
```

* Como vamos definir o tipo dos retorno no nosso código? usando `@spec`

```elixir
@spec soma(number(), number()) :: String.t()
def soma(a, b), do: "O resultado é #{a+b}"
```

* A sintaxe é algo como `nome_da_funcao(tipo_do_argumento_1, ... tipo_do_argumento_n) :: TipoDoRetorno` (a lista de tipos ta linkada logo abaixo)

* Okei. Coloquei no meu codigo, passei o tipo errado e ele ainda sim funciona. Ué?

* Bom, elixir é uma linguagem dinamicamente tipada e não verifica por tipos na hora de compilar, então tudo vai funcionar tranquilo e favorável

* Ok, mas porque eu especifique tipo se elixir nem faz essa verificação?

* Primeiro porque é uma informação a mais pra ajudar a entender o código, e depois porque existe uma ferramenta que você pode colocar no seu projeto que faz a verificação real oficial dos tipos do seu código. O nome dela é Dialyzer

* Beleza :D Achei top! mas se eu criei um tipo novo? como uma struct, por exemplo? como eu defino isso?

* Vamos voltar pro exemplo do mercadinho

```elixir
defmodule Mercadinho.Padaria.Bolo do
  defstruct sabor: nil,
            preco: 0, 
            cobertura: false,
            pronto: false 

  @type t::%Mercadinho.Padaria.Bolo{
    sabor: String.t,
    preco: non_neg_integer, 
    cobertura: boolean(),
    pronto: boolean() 
  }
end
```

* Okei, agora eu defini o tipo de tudo. Mas vamos incrementar o código mais um pouco.

* Agora vamos criar uma função que assa bolos

```elixir
def assar_bolos(bolos), do: Enum.map(bolos, fn x -> "O bolo #{x.sabor} está assando" end)
```

* Ela recebe uma lista de bolos e retorna uma lista de strings.

```elixir
alias Mercadinho.Padaria.Bolo # Isso ta aqui pra simplificar a tipagem

@spec assar_bolos([Bolo.t]) :: [String.t()]
def assar_bolos(bolos), do: Enum.map(bolos, fn x -> "O #{x.sabor} está assando" end)
```

* Se a gente tivessse usado o alias `alias Mercadinho.Padaria.Bolo`, o tipo seria só `[Bolo.t])`

* (para saber todo os tipos, [aqui tem a lista](https://hexdocs.pm/elixir/typespecs.html#built-in-types))
