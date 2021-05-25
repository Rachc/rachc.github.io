---
layout: post
title:  Aprendendo programação funcional com elixir - cap1
date:   2021-02-09 08:00:00 -0300
image:  functional01.jpg
tags:   Funcional, Elixir
---
# Disclaimer
* Os posts dessa série são minhas anotações pessoais dos meus estudos do livro "Aprendendo programação funcional com elixir.

* Eles não passaram por revisão, possuem typos e alguns erros. 

* Estão mais próximos de pensamentos desconexos do que de um texto estruturado.

* Dito isso, segue anotações (: 

# Intro
Pra quem chegou agora eu, junto com a Elaine Watanabe e a Juliana Helena, estamos estudando o livro Learn Functional Programming with Elixir do autor Ulisses Almeida.

Estamos fazendo lives [aqui](https://www.youtube.com/channel/UCHDwILYk5-LpMAUIUcPSZMw), e essas são as minhas anotações (:

O link da live do primeiro capítulo, referente às essas notas [está aqui](https://www.youtube.com/watch?v=DYszf5MF8fA)

# Cap.01 - Por que funcional?
* Um paradigma de programação consiste em regras e principios de design para a construção de um software. 
* Na hora que a gente sente que as pessoas estão migrando de paradigma, significa que algo grande está acontecendo.
* Hoje nós precisamos processar umas várias coisas ao mesmo tempo, e os CPUS não estão ficando mais rápidos, porém estamos com mais cores e threads, o que significa que precisamos tirar proveito disso.
* Nós precisamos escrever código que tire proveito disso, de forma que ele consiga rodar nessas várias threads nos multicore da vida.
* Infelizmente, linguagens imperativas ou orientadas a objeto possuem alguma limitação que fazem com que seja mais difícil trabalhar sob essas demandas multi-core.

## As limitações de uma linguagem imperativa
* Linguagens imperativas compartilham de valores que sofrem mutação, o que significa que várias partes de um programa podem referenciar um valor e modificá-los, o que pode causar uma dor de cabeça na hora de debugar, já que acreditamos que estamos falando do valor "x", mas um outro ponto do programa ele foi modificado pra "q".
Segue exemplo:
```ruby
doces = ["chocolate", "sorvete", "pastel de belem", "sonho"]
doces.pop
# => "sonho"
doces.push("chiclete")
# => ["chocolate", "sorvete", "pastel de belem", "chiclete"]
puts doces.inspect
# => ["chocolate", "sorvete", "pastel de belem", "chiclete"]
```

* No exemplo acima nós conseguimos modificar os dados adicionando ou removendo itens.
* Agora imagina se várias partes de uma aplicação grande está rodando em paralelo e mexendo nos nossos doces?
* O que acontece se um valor muda no meio da operação por conta de outro processo? É bem difícil de prever.
* Muitas linguagens não funcionais oferecem uma série de libs que ajudam a resolver esse problema, porém linguagens funcionais já vem com isso pronto desde da sua concepção

## Migrando para a programação funcional
* No paradigma funcional, funções são a nossa principal ferramenta de construção de código (da forma que em orientação a objeto, objetos que são o ponto mais importante).
* Ao procurar por programação funcional nos buscadores, nós esbarramos em uma série de termos que parecem que foram feitos por matemáticos, e isso faz parecer que programação funcional tem uma barreira de entrada muito maior do que realmente é
* (agora o ulisses fala rapidamente, em um paragrafo, sobre Lambda, funções anônimas e funções como cidadão de primeira classe)
* Elixir é uma linguagem funcional com uma linguagem fácil e acessivel, porém construida em cima do ecossistema erlang, que possui mais de 30 anos e 9s de confiabilidade
* Quando você escreve código funcional em uma linguagem funcional, você tem código que vive de forma harmoniosa, mas para que você consiga escrever um código como deveria ser, é importante entender os conceitos principais, como **Imutabilidade, funções e código declarativo**

# Trabalhando com Dados imutáveis
* Linguagens convencionais precisam de mecanismos para travar seus dados para trabalhar com concorrencia e paralelismo, mas em elixir, que é funcional, não é preciso, graças a imutabilidade
* Executando as mesmas operações acima com elixir:
```elixir
doces = ["chocolate", "sorvete", "pastel de belem", "sonho"]
List.delete_at(doces, -1)
# => "sonho"
doces +++ ["chiclete"]
# => ["chocolate", "sorvete", "pastel de belem", "sonho", "chiclete"]
IO.inspect doces
# => ["chocolate", "sorvete", "pastel de belem", "sonho"]
```
* Da pra ver que nenhuma operação passa pra frente a alteração da lista de doces original.
* Como nós não temos mutação da lista original, nós podemos rodar esse comando em paralelo, em qualquer ordem, que não vai quebrar
* Escrever funções simples nos ajudam a usar o paralelismo de forma top
* Podemos pensar que sem toda essa mutabilidade, gerar novos valores é mais lento, mas na real elixir tem umas estruturas de dados que faz o reuso de valores na memória, fazendo que tudo isso aconteça de forma muito eficiente
* Até temos imutabilidade em outras linguagems não funcionais (como o ".freeze" em ruby), mas muitas vezes esses recursos não funcionam tão bem como funcionariam em uma linaguagem funcional.
* Usando o exemplo do freeze em ruby em uma lista, ele nos impede de adicionar valores novos, mas podemos alterar valores existentes
* Se você está usando uma linguagem que tem mutabilidade como padrão, é fácil esbarrar em erros.

# Construindo programas com funções
* Não é possível criar um programa sem funções dentro do paradigma funcional
* Combinamos várias pequenas funções para criar programas grantes e complexos.
* Conseguimos diminuir a complexidade dos nossos códigos se as funções tem as seguintes propriedades:
	* Valores imutáveis
	* O resultado de uma função é influenciado apenas pelos argumentos das funções
	* Os efeitos causados por uma uma função está limitado apenas o valor que ela retorna.

* Funções que possuem as propriedades acima são chamadas de funções puras (mas sabemos que as coisas nunca são simples assim, e que também existem função impura)
* Um exemplo de função pura é uma função de soma:
```elixir
add2 = fn(n) -> n+n end #<--- Olha a função anonima aiii
add2.(2)
# => 4
```

## Usando valores explicitamente
* Em programação funcional os valores são passados explicitamente entre funções, deixando claro pro desenvolvedor o que está acontecendo
* Já em orientação a objeto, os objetos possuem estados, e todas as funções/métodos existentes estão intimamente ligados aos estados.
* Se modificarmos um estado de um objeto, o resultado da função pode ser outro diferente.
* Isso gera uma dependenia complexa entre métodos e estado, o que ocasionalmente pode ser difícil de debugar e manter (especialmente se a gente não aplica muito bem os principios de solid, fazemos modificações indevidas, deixamos os objetos muito complexos)
* A gente precisa estar sempre muito disciplinado pra estar sempre apliacando boas práticas
* (observação de Rachel aqui: não é que OO é ruim porque tem estados e FP é bom porque não tem. O ponto acima é que temos uma camada a menos na complexidade da construção de software. Embora não tenhamos estados, da pra fazer código funcional bem cheio de dependencias)
## Usando funções como argumentos
* Funções são tão inerentes da programação funcional, que é muito comum usar funções como argumentos de funções
```elixir
iex> Enum.map(["verde", "azul", "roxo"], &String.upcase/1)
["VERDE", "AZUL", "ROXO"]
```
* No exemplo acima nós temos um map que recebe 2 argumentos: o enum que queremos que seja modificado e a função que vai modificar ele (no caso `String.upcase/1`)
## Transformando valores
* Em elixir temos um fluxo de transformação (que é a coisa mais linda do mundo). Nos temos um operador especial chamado pipe `|>` que combina multiplas funções
* Então no lugar de ter:
```ruby
def capitalize_words(title) do
  join_with_white_space(
    capitalize_all(
      String.split(
        title
      )
    )
  )
end
```

* nós podemos ter:
```elixir
def capitalize_words(title) do
  title
  |> String.split
  |> capitalize_all
  |> join_with_white_space
end
```

E dai o código fica muito mais fácil de entender
* (Rachel aqui: Funções são peças que podem ser encaixadas nas funções, como se fosse plug'n'play, mas devemos nos atentar a assinatura das funções. Se a primeira função produz um string, a segunda deve receber uma string (e se ela produzir um boolean, a terceira tem que aceitar um boolean, e por ai vai)

# Declarando código
* Programação imperativa foca em "como vamos resolver um problema"?
* Enquanto na programação funcional (rachel: mas não apenas), o foco é em responder "o que é necessário para resolver um problema"
* Como exemplo de função imperativa nós temos o código abaixo, que possui loops e você precisa informar com detalhes como que as coisas devem acontecer

```javascript
var pratos = ["camarão", "macarrão", "batatas"]

function upcase(lista) {
  var novaLista = [];
  for (var i = 0, i < lista.length, i++) {
    novaLista.push(list[i].toUpperCase());
  }
  return novaLista;
}

upcase(pratos);
// => ["CAMARÃO", "MACARRÃO", "BATATAS"]
```

* Já em programação funcional nós trocamos loops por recursão e focamos mais em como nós gostariamos de tratar o dado do que em como nós gostaramos de lidar com ele

```elixir
defmodule StringList do
  def upcase([]), do: []
  def upcase([first|rest]), do: [String.upcase(first)|upcase(rest)]
end
```

* (Rach aqui: as entranhas dessas função a gente vai entender mais pra frente. No momento é melhor entender do que sentir, mas nós temos no exemplo acima formas de manipular listas, recursão e pattern match. No exemplo acima é como se usassemos "if" com esteroides)
* Podemos também usar algumas funções prontas como 

```elixir
var pratos = ["camarão", "macarrão", "batatas"]
Enum.map(pratos, &String.upcase/1)
# => ["CAMARÃO", "MACARRÃO", "BATATAS"]
```

* Hoje muitas linguagens abraçaram a forma declarativa de se fazer código. Enuns estão em todos os lugares.

# Créditos 
Anotações feitas a partir do estudo do livro [Learn functional programming with elixir](https://amzn.to/3cVX6tm) do Ulisses Almeida

[imagem da capa por Hans Reniers](https://unsplash.com/photos/lQGJCMY5qcM?utm_source=unsplash&utm_medium=referral&utm_content=creditShareLink)
