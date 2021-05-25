---
layout: post
title:  Aprendendo programação funcional com elixir - cap7
date:   2021-05-13 08:00:00 -0300
image:  functional01.jpg
tags:   Funcional, Elixir
---
# Disclaimer

* Os posts dessa série são minhas anotações pessoais dos meus estudos do livro "Aprendendo programação funcional com elixir.

* Eles não passaram por revisão, possuem typos e alguns erros. 

* Estão mais próximos de pensamentos desconexos do que de um texto estruturado.

* Dito isso, segue anotações (: 

# Intro

* Existem funções que podem retornar resultados diferentes com o mesmo input
* Se temos uma função que espera um numero mas recebe um texto? E se temos uma função que busca dados de um banco de dados?
* Algo que se espera de todo programa é ter que lidar com erros. 
* Trabalhar sem estratégia nenhuma gera código difícil de manter.
* A melhor estratégia pra se ter uma base de código saudável é identificar e isolar as partes que possuem resultados inexperados
* Em elixir temos 4 formas de lidar com isso (meio que 3.5, na verdade, porque uma das formas envolve instalar bibliotecas externas)
	* estruturas condicionais -> case e if
	* Try -> Da forma que pessoas familiarizadas com java estão acostumadas
	* Monads -> bastante usado em linguagens funcionais que possuem tipagem estática
	* With -> Uma diretiva diferenciada que combina pattern match com estrutura condicional

# Funções puras vs Impuras

## Funções puras

* Quando você não consegue prever o resultado de uma função, ela é impura
* ` def total(a, b), do: a + b` é uma função pura, pois se você passar `total(1, 2)` sempre vai retornar 4. 
* mas como assim tem caso que você vai ter uma função que você passa um argumento e o resultado vai ser diferente toda vez? existe isso? (foi o que eu pensei a principio, pois não tenho criatividade XD)

## Funções Impuras

* Imagina que você está buscando um dado em um banco de dado.
* (vamos usar bolos nesses exemplos? vamos)
* você vai lá, passa o ID 42 e retorna um bolo de limão (`busca_bolo(42)`)
```
 %Bolo{sabor: "bolo de limão", preco: 30, cobertura: true, pronto: false, id: 42}
```
* Agora pensa que em algum momento alguém atualizou o bolo de limão para que ele tivesse cobertura. O que acontece quando você for procurar um bolo de id 42? E se alguém decide que não quer mais vender bolo de limão e apagar do banco de dados? O resultado vai ser diferente da primeira vez. 
* Então a função é a mesma, passamos o mesmo argumento, mas o resultado pode vir diferente.
* Uma função que busca no banco de dados é um exemplo de função impura, mas temos outras, como por ex uma função que gere um numero randomico, ler e escrever um arquivo, acessar API, ou até um `DateTime.utc_now()`

### Identificando funções impuras

* Uma definição de função impura é: Funções que são afetadas por valores fora do escopo.
```elixir
> multiplicador_cobertura = 1.5
> def valor_do_bolo(valor), do: valor * multiplicador_cobertura
> valor_do_bolo(10)
15
> multiplicador_cobertura = 2
> valor_do_bolo(10)
20
```
* O multiplicador do valor de um bolo com cobertura é um valor externo. Porém, graças a imutabilidade, podemos alterar o multiplicador o quanto quisermos que o resultado vai ser sempre o experado. isso significa que essa função é pura ou impura? É pura :D
* Outra definição de função impura são funções que possuem efeitos colaterais. Efeitos colaterais podem incluir escrever mensagens noterminal, modificando um estado global ou inserindo/buscando items no banco de dados
```elixir
def valor_do_bolo(valor) do
  total = valor * multiplicador_cobertura
  IO.puts(total)
end
```
* O valor do bolo ta ali na primeira linha da função, porém a gente da uma bagunçada no coreto printando essa mensagem. (e se quisermos usar a função do calculo do valor do bolo em outro lugar, sem printar o resultado na tela? não é possível)
* Funções que se comportam dessa forma é uma má prática. I ideal seria ter uma função para calcular o valor do bolo e depois, fora dela, printar o valor
* A melhor prática de desenvovimento é isolar as funções puras das impuras
* O nome "função impura" pode dar a impressão que elas são algo a serem evitadas, mas não é verdade. Um software útl precisa ter funções impuras (se não tiver, nunca poderiamos fazer alterações no banco de dados, por exemplo)
* Porém as funções puras são mais fáceis de manter
* Então em um mundo ideal teriamos um software com mais funções puras  com funções impuras isoladas
* Ok! mas como eu isolo uma função impura?

# Controlando o fluxo das funções impuras

## case/if
* A primeira estratégia coberta no capitulo 3, sobre controle de fluxo usando estruturas condicionais
* É ótimo para lidar com casos simples, mas casos mais complexos podem ficar confusos
```elixir
defmodule Shop do
  def checkout(price) do
    case ask_number("Quantity?") do
	   :error -> IO.puts("It's not a number")
	   {quantity, _} -> quantity * price
	end
  end
  
  def ask_number(message) do
   message <> "\n"
   |> IO.gets
   |> Integer.parse
  end
end
```
* No caso a cima estamos lidando com um imput externo, então o resultado pode vir qualquer coisa. 
* Como precisamos que o imput seja um numero, precisamos tratar pra no caso de vir outra coisa
* Usar if/case pode virar um grande emaranhado de ifs e cases aninhados. 
```elixir
def checkout() do
    case ask_number("Quantity?") do
	   :error -> IO.puts("It's not a number")
	   {quantity, _} -> 
	     case ask_number("Price?") do
		   :error -> IO.puts("It's not a number")
		   {price, _} -> quantity * price
		 end
	end
  end
```
* Podemos trocar tudo isso por um pattern match pra ficar mais suave
```elixir
def checkout() do
  quantity = ask_number("Quantity?")
  price = ask_number("Price?")
  calculate(quantity * price)
end

def calculate(:error, _), do: IO.puts("It's not a number")
def calculate(_, :error), do: IO.puts("It's not a number")
def calculate({quantity, _}, {price, _}), do: quantity * price
```

* Control flow é uma solução simples, bastante legível e amigável, mas quando seu código se torna complexo, pode ser dificil a manutenção.
* O ideal é optar por ela apenas quando for algo simples. Caso você perceber que está ficando complexo, o ideal é refatorar pra uma das opções abaixo

## Try, rescuing, catching 

* Com algumas funções você tem pouco controle do código. As vezes nos deparamos com erros por ai e é interessante capturalos antes que ele chegue no usuário
* Em elixir nós conseguimos facilmente identificar que funções retornam um erro porque elas terminam com `!`
* O que acontece com try/catch/rescue é que nós encapsulamos o código que pode apresentar problema em um bloco `try` e no caso dele der erro, ele cai no `rescue` (ou no `catch`, depende do que você for usar)
* Em funcional não é comum nós jogarmos erros diretamente na cara do usuário, então é incomum usarmos esse artificio no nosso código, porém nada impede de esbarrarmos em libs que explodem erro na cara do usuário, então é interessante aprender como podemos lidar com esse tipo de erro.

### Try/Rescue

* usando o mesmo exemplo acima, podemos mostrar como ele seria usando um bloco de try/rescue
```elixir
def checkout() do
  try do # <- Aqui encapsulamos o caminho feliz do código
    {quantity, _} = ask_number("Quantity?")
	{price, _} = ask_number("Price?")
	quantity * price
  rescue # <- E se não for feliz, ele retorna o erro abaixo
    MatchError -> "It's not a number"
  end
end
```
### Try/Trow/Catch

* `Throw` e `Catch` são bem parecidos com `raise` e `rescue`. A diferença é que `Throw/Catch` não necesariamente precisa ser um erro gerado pelo sistema.
```elixir
@invalid option {:error, "Invalid Option"}
def parse_answer(answer) do
  case Integer.parse(answer) do
    :error -> throw @invalid option # <--- Aqui nós estamos criando uma tupla pra representar um erro
	{option, _}
  end
end
################
def something(pastel) do
  try
    pastel
    |> funcao_marota_1
    |> parse_answer # <--- nossa função que não retorna erro, mas uma tupla
    |> outra_funcao_marota_aqui
  catch
    {:error, message} -> # Pattern match maroto na tupla de erro que criamos
	  display_error(message)
  end
end
```
* A maior vantagem do try/catch é a maior visibilidade do caminho feliz da nossa aplicação
* Mas ela torna nossa função mais dificil de reutilizar por conta das features extras da linguagem. O aumento da complexidade é o que faz com que nós, desenvolvedores elixir, evitemos.

## Monads

* Monads são bastante usadas em outras linguagens funcionais, como Haskell
* Em elixir monads não vem naturalmente implementados na liguagem, então é preciso que você instale alguma biblioteca externa e adicione mais essa complexidade ao seu código
* (Não entrei em muito detalhe sobre o assunto porque não é muito comum, mas pra quem tiver curiosidade vale dar uma olhada em libs como [MonadEx](https://github.com/rob-brown/MonadEx), [Towel](https://github.com/knrz/towel))

## With

* Em elixir temos a opção de usar `With` que combina várias clausulas + pattern match
* Uma coisa que é interessante reparar é o uso do `<-` (normalmente nós usamos `->`)
* Com with nós usamos pattern match para apontar o que queremos caso o resultado seja positivo, e jogamos um else caso não
```elixir
def checkout() do
    with {quantity, _} <- ask_number("Quantity?"),
        {price, _} <- ask_number("Price?") do
    quantity * price
  else
    :error -> IO.puts("It's not a number")
  end
end
```
* A maior vantagem do `with` é sua flexibilidade. Da pra combinar pattern match, sem novas lis, conceitos ou estrutura de dados, além da legibilidade
* O problema é que não é compativo com pipe operators
* Na maior parte dos casos é a nossa primeira opção