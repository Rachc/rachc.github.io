---
layout: post
title: Programando phoenix - Cap 2 - Como funciona por debaixo dos panos?
date: 2021-05-13 08:00:00 -0300
image: phoenix.jpg
tags: Funcional, Elixir
---

## Disclaimer
- Os posts dessa série são minhas anotações pessoais dos meus estudos do livro "Aprendendo programação funcional com elixir.
- Eles não passaram por revisão, possuem typos e alguns erros.
- Estão mais próximos de pensamentos desconexos do que de um texto estruturado.
- Dito isso, seguem anotações (:

# capitulo 2 - The Lay of the land
- Você pode pensar em uma requisição como uma grande função que recebe uma url como parâmetro e retorna um site.
- Neste capítulo nós vamos entender um pouco o que acontece no phoenix entre enviar a url até renderizar o conteúdo na tela.

## Funções simples
- Aqui ele explica por cima como funcionam os pipes.
- Com um resumo bem resumido, pipes são sequências de funções.

```elixir
func3(func2(func1("pastel")))
```

- vira.

```elixir
"pastel"
 |> func1() # vai receber pastel como primeiro argumento e vai cuspir algo
 |> func2() # vai receber esse algo como 1 argumento, transformar e cuspir outro algo
 |> func3() # vai pegar outro algo como primeiro argumento, transformar e nos dar o resultado final
```

- Em Phoenix nós temos a entidade `connection`, que é um pacotinho que chega do usuário com várias informações (qual requisição ele fez? ele passou algum parâmetro? o que tem no cabeçalho da requisição).

- O phoenix pega esse pacotinho, faz uma sequência de transformações até chegar no resultado final, que é a página renderizada.

- Pensando que um request é uma grande função que pega sua conexão, transforma ela aos poucos e renderiza pro usuário, no phoenix conseguimos ver essas transformações através de pipes.

```elixir
connection
  |> endpoint()
  |> router()
  |> pipelines()
  |> controllers()
```

- Neste capítulo toda essa sequência de pipes é explicado com detalhe, mas conseguimos resumir aqui um pouco.
  - o **endpoint** é a porta de entrada. É a identificação de onde o usuário quer acessar. é um "www.pastel.com/queijo", por exemplo.
  - Com esse endpoint em mãos, ele vai pro **router**, que funciona como uma espécie de mapa. Ele vê que você acessou a página "queijo" e vai identificar onde está o código que executa a ação da página "queijo".
  - antes de mandar o código pro **controller**, que é onde está a ação, você passa por uma **pipeline** de transformações. É nesse passo que vamos saber se a função vai retornar uma página pro usuário, ou se por acaso estamos lidando com uma API e retornaremos só um JSON por ex.

## Inside controllers
- O phoenix também recomenda uma arquitetura MVC -> Model, view e controller (e template :P).
- O model é a camada que conversa com o banco de dados, o controller faz o tratamento desses dados para chegar na view da forma que a gente espera, enquanto a view é responsável pela exibição desses dados (muitas vezes organizar os templates de uma forma que nós esperamos).
- O controller do phoenix também segue uma timeline própria.

```elixir
connection
 |> controller()
 |> common_services()
 |> action()
```

- Sendo os **common_services** plugs (que vamos falar mais dele, mas pense em plugs como uma sequência de modificações feita por algumas bibliotecas). As suas **actions** podem variar um pouco. As vezes é autenticar um usuário, às vezes acessar o banco de dados ou renderizar a view
- Aqui é um exemplo de como o pipeline de um action pode parecer:

```elixir
connection
  |> find_user()
  |> view()
  |> template()
```

- No código acima deu pra ver que a **connection** chega pra gente (depois de já ter passado pelo pipe acima e ter sofrido algumas transformações), executa alguma função de busca, pasa o resultado pra **view** que organiza as informações a serem enviadas, e passam pra frente para o **template** (que pode ser um arquivo html ou json ou outra coisa) que é compilado de uma forma que o elixir entenda. (muitas as vezes o arquivo da view pode combinar mais de um template existe para chegar em um resultado final).
- Vale frisar que todas essas ações não vão aparecer pra você como um único pipe. Essas funções estão espalhadas pela aplicação de uma maneira que faça sentido.
- Pode parecer confuso à primeira vista, mas depois que você se acostuma, faz sentido. Manter separado é bom pra mantenabilidade.

## Instalando o phoenix
- Em alguns dias deve sair um guia bonitinho sobre como instalar elixir no windows, mas vamos deixar aqui o resumão.
- Pra instalar o Phoenix precisamos primeiro instalar o elixir, e para instalar o elixir nós precisamos instalar o erlang.
- Costumo a usar o asdf para instalar as linguagens necessarias -> [documentação, em inglês do asdf](https://asdf-vm.com/guide/getting-started.html#_1-install-dependencies).
- Vamos precisar também instalar o NodeJs, porque a pasta de assets do node precisa.
- Precisamos instalar o Postgresql, pra poder usar o ecto e no final de tudo isso, conseguimos instalar o phoenix. Então vamos facilitar aqui e por uma ordem pra vocês conseguirem instalar e rodar seu phoenix :D
  1. [asdf](https://asdf-vm.com/guide/getting-started.html#_1-install-dependencies)
  2. Erlang (com asdf)
  3. Elixir (com asdf)
  4. Nodejs (com asdf)
  5. Postegresql
  6. Phoenix
- O comando para instalar o phoenix é:

```elixir
mix archive.install hex phx_new
```

- Pra saber se deyu tudo certo, rodamos `mix phx.new -v`

## Criando um projeto teste
- Agora vamos pôr a mão na massa e criar um projeto :D Assim vamos não só descobrir como criar, mas ter algo concreto que podemos olhar e identificar tudo que foi falado acima. Vamos criar um projeto chamado "ola". O comando para criação de um novo projeto phoenix é o seguinte:

```bash
mix phx.new ola
```

- Sendo ola o nome do projeto.
- Ele vai perguntar se você quer instalar as dependências (a resposta é y, de yes).
- Agora você precisa criar seu banco de dados rodando um.

```bash
mix ecto.create
```

- e toda vez que você for rodar a sua aplicação web, você digita.

```bash
mix phx.server
```

- e você também pode rodar seu servidor dentro do iex com o segundo comando (é ótimo pra debugar).

```bash
iex -S phx.server
```

- Se você entrar na pasta do seu phoenix agora, já vai conseguir ver um bocado de arquivos e diretórios /o/

## Criando uma feature

- Nossa primeira feature vai ser simples. Vamos criar uma página que tem a string "Olá!" escrita.
1. O primeiro passo é ir no arquivo `router`, localizado na pasta `lib/NOME_DA_SUA_APLICACAO_web/router.ex`.
2. Achar esse código aqui para adicionar a nossa rota e adicionar uma nova rota.

```elixir
  scope "/", SuaAplicacaoWeb do
    pipe_through :browser

    get "/", PageController, :index
  end
```

```elixir
  scope "/", SuaAplicacaoWeb do
    pipe_through :browser

    get "/ola", OlaController, :mundo ####### <----- Essa é a linha nova.
    get "/", PageController, :index
  end
```

- Agora a nossa rota `ola` já existe :D o problema é que não tem nada ainda. Se acessarmos a página agora, vamos esbarrar em um erro do tipo "undefinedFunctionError". Lendo com calma, podemos descobrir que ele avisa que não temos controller, então vamos criar.
- Quando esbarramos em um problema na nossa aplicação, a página de erro do Phoenix é muito boa pra tentarmos entender o que não está acontecendo.

3.  Agora que temos uma rota apontando para um controller, precisamos criar o controller.
    - Os controllers também ficam na página web da sua aplicação. então criaremos um novo na pasta `lib/NOME_DA_SUA_APLICACAO_web/controllers/nome_do_seu_controller.ex` (no caso da nossa aplicação ele é o `ola_controller.ex`).
    - seu controle deve ter +- essa cara:

```elixir
defmodule SuaAplicacaoWeb.OlaController do
  use SuaAplicacaoWeb, :controller #não esquecer

  def mundo(conn, _params) do ### <- atenção no nome da função
    render(conn, "mundo.html")
  end
end
```

- O nome da função `mundo` no arquivo acima é a mesma que você definiu no seu controller.
  - Como não temos grandes coisas a fazer no controller, só estamos mandando renderizar a página `mundo.html` (que ainda não foi criada).
- Reparando que nós estamos recebendo a connection do usuário e estamos passando pra frente também.
- Se tentarmos entrar no caminho `/ola` na nossa aplicação ele vai continuar dando erro, porém o erro deve ser outro :D

4. Agora criamos a view na pasta `lib/NOME_DA_SUA_APLICACAO_web/views/ola_view.ex`.
   - Reparou que o nome da sua view é o mesmo do seu controller? Um controller vai procurar pela view do mesmo nome.
   - Como não temos nada muito complicado na nossa view, ela é bem simplinha.

```elixir
defmodule SuaAplicacaoWeb.OlaView do
  use SuaAplicacaoWeb, :view
end
```

- Mais uma vez, se você entrar no localhost, vai aparecer um erro, porém um novo erro /o/

5. O último passo é criar um template pra nossa função :D Ele deve ter o mesmo nome que definimos no controller. No nosso caso, ele se chamará `mundo.html.eex` e ficará na pasta `lib/NOME_DA_SUA_APLICACAO_web/templates/NOME_DA_SUA_APLICACAO/mundo.html.eex`.
   - a extensão `eex` significa que podemos acrescentar código elixir no nosso template :D (com alguns ajustes, que mostraremos já).
   - Esse arquivo deverá ser +- assim.

```elixir
<h1>do template: "Olá Mundo :D"</h1>
```

- Agora se formos na nossa página você possivelmente já consegue ver a página que nós criamos :D
- Nós temos live reloading, então a página é atualizada automaticamente a cada modificação :D
- Agora você já consegue criar uma página web simples :D
- Vou deixar aqui uma colinha pra facilitar sua vida.

### Colinha básica para desenvolvimento de uma funcionalidade

- Colocar ela na rota, na pasta `lib/NOME_DA_SUA_APLICACAO_web/router.ex`, apontando para o controller certo (atenção no nome das funções que você vai chamar).
- Cria o controller na pasta `lib/NOME_DA_SUA_APLICACAO_web/controllers/nome_qualquer_controller.ex`
- Cria a view na pasta`lib/NOME_DA_SUA_APLICACAO_web/views/mesmo_nome_qualquer_view.ex`
- Criar o template na pasta `lib/NOME_DA_SUA_APLICACAO_web/templates/NOME_DA_SUA_APLICACAO/nome_qualquer.html.eex`

### Usando parâmetros na rota

- O que acontece se a gente quiser dar um olá de forma dinâmica, que mude de acordo com o que a gente digite na url?
- primeiro vamos na rota e depois adicionamos o parâmetro que nós queremos.

```elixir
get "/ola/:nome", OlaController, :mundo ####<- Agora temos uma parâmetro :nome pra ser usado
```

- Depois mudamos o controller para que ele consiga capturar e usar nosso novo parâmetro.

```elixir
defmodule SuaAplicacaoWeb.OlaController do
  use SuaAplicacaoWeb, :controller #não esquecer

  def mundo(conn, %{"nome" => var_nome}) do ### <- antes aqui era _params
    render(conn, "mundo.html", nome: var_nome) ## Passamos aqui também
  end
end
```

- Conseguimos capturar a variável através do pattern match :D e passamos pra frente em uma lista de chave e valor (se tivéssemos mais de um parâmetro passaríamos assim: [par1: par1, par2: par2]).
- Sabemos que podemos usar chave e valor de duas formas. A primeira é usando um atom como chave, e ele aparece assim `chave: valor` e a segunda é usar `"chave" => valor`. Até podemos usar um atom ali na função mundo, porém o conselho é usar string, já que o valor que está chegando pra nós é algo externo. Nesse caso a conversão de atom para string é automática, porém como atoms não são coletados no garbage collector, é melhor deixar pra usá-los apenas quando é um valor usado na aplicação.
- OKEI! AGORA VAMO PÔR NO TEMPLATE.
- Lembra que eu comentei que poderíamos usar código elixir nos arquivos de extensão `.eex`? agora chegou a hora de brilharmos.
- Vamos por no template a variável `nome` que criamos :D

```elixir
<h1>do template: "Olá <%= String.capitalize(@name) => :D"</h1>
```

- em um arquivo `eex` podemos usar os símbolos `<%= =>` como se fosse uma tag HTML e por código elixir no meio (:
- Agora se acessarmos `localhost:4000/ola/pastel` devemos ver a mensagem "Olá Pastel :D".

## Entendendo as outras coisas mais da nossa aplicação

- No começo do capítulo nós falamos sobre como nossa aplicação phoenix era uma sequência de funções. Agora a pouco nós entendemos como funciona o pipe do controller até chegar na view, mas ainda não entendemos o que acontece ali no meio. O que é plug? O que são esses arquivos todos que tem ai? Como eu mexo na configuração do meu projeto? e como eu instalo uma dependência? Daqui pra frente vamos explicar isso.
- Para começar, podemos dizer que "plugs" são funções que são "plugadas" na nossa aplicação e, uma por vez, vai modificando a conexão do nosso usuário para que fique do jeito que queremos.
- Okei, agora que já entendemos +- o que é plug, podemos passar pra frente e entender o resto da aplicação. Voltaremos a falar de plug várias vezes, então vai ficando mais fácil de entender.

### A estrutura de arquivos do phoenix

- Conseguimos identificar na nossa árvore de arquivos as seguintes pastas:

  - assets
  - config
  - deps
  - lib
    - sua_aplicacao
    - sua_aplicacao_web
  - test

- Na pasta de `assets` estão os arquivos estáticos, de css e JS.
- Em `config` nós temos as configurações, árvores de supervisão e tudo mais.
- `deps` são as dependências do nosso projeto. Então se você precisar entender o código de alguma lib que você instalou, ela vai estar aqui.
- Os arquivos que você vai criar pro seu projeto estarão em `lib`. Os que têm relação direta com a web, como controllers, views e templates, vão na pasta `sua_aplicacao_web`, e os que relação com lógicas de negócio, processos, jobs e afins irão em `sua_aplicação`.
- já na pasta `tests` vão estar seus testes (:

### Config :D

- Phoenix são aplicações phoenix, e como toda a aplicação elixir, nós temos as mesmas pastas (e mais algumas).
- `lib`, `test` e os arquivos `mix.exs` e `mix.lock` fazem parte da estrutura padrão que vem no phoenix.
- temos arquivos `.ex`, que são compilados pela beam, e temos também arquivo `.exs`, que não são compilados pela beam, mas em memória, cada vez que eles são executados.
- No arquivo `mix` estão alguns informações como dependências e outras configurações.
- o `mix.lock` tem a lista de todas as dependências do nosso projeto com as versões usadas. Isso garante que em todas as máquinas que o projeto rode, as dependências sejam as mesmas.
- Temos também um arquivo chamado `application.ex` em `lib/NOME_DA_SUA_APLICACAO` , que vai conter informações como árvores de supervisão. Muitas vezes você vai instalar alguma lib e precisa mexer aqui para que as configurações funcionem perfeitamente.

#### Environments e Endpoints

- Dentro da pasta config nós temos alguns arquivos de configuração. Existe um arquivo principal, chamado `config.exs` que tem as configurações em comum de todos os ambientes, bem como `dev.exs`, `prod.exs` e `test.exs`, com as configurações específicas de cada ambiente.
- Temos também um arquivo chamado `prod.secret.exs` que é responsável por carregar secrets e valores de config dos ambientes.
- Também podemos trocar entre os diversos ambientes usando a variável `MIX_ENV=` (como `MIX_ENV=test`, por exemplo)
- no arquivo config nós temos um endpoint +- com essa cara.

```elixir
use Mix.Config

#Configure the endpoint
config :sua_aplicacao, SuaAplicacaoWeb.Endpoint,
  url: [host: "localhost"],
  (...)
```

- Ele é o limite entre a conexão e a nossa aplicação web.
- Mesmo sem entender muito do código, da pra ver que ele é uma função `config`, que chama `SuaAplicacaoWeb.Endpoint`, da sua aplicação `:sua_aplicacao`.
- Se olharmos o arquivo `SuaAplicacaoWeb.Endpoint` vamos ver um bocado de plug, que nada mais são que funções.

#### Rotas :D

Agora que já conseguimos entender os endpoints, podemos dar uma olhada no arquivo das rotas novamente

```elixir
  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_flash
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end

  pipeline :api do
    plug :accepts, ["json"]
  end

  scope "/", SuaAplicacaoWeb do
    pipe_through :browser ### <------ definimos no escopo qual pipe queremos

    get "/", PageController, :index
  end
```

- No arquivo de rotas você pode definir algumas várias pipelines para fazer todo tipo de modificações. Por exemplo, no arquivo de rotas que tem por padrão tem duas pipelines. Uma que vai definir as rotas que serão acessadas pelo navegador, e as rotas que serão acessadas via API.
- Você pode acrescentar quantos pipes você quiser. A forma mais comum de personalização de pipe é para autenticação. E se você quiser que parte de uma rota seja acessível a todas, outra só pros usuários cadastrados e outra só para o admin? É no pipe que você faz essa definição.
- Depois de criado/definido qual pipe você vai usar, você cria um novo escopo e define quais rotas usarão aquele pipe.

## Revisitando os pipes agora que sabemos onde estão os arquivos

* Pipe geral:

```elixir
connection         # Plug.com
  |> endpoint()    # lib/sua_app_web/endpoit.ex
  |> router()      # lib/sua_app_web/router.ex
  |> pipelines()   # lib/sua_app_web/router.ex
  |> controllers() # lib/sua_app_web/controllers/*
```

* Pipe dos controllers:

```elixir
connection
 |> controller()      # lib/sua_app_web/controllers/*
 |> common_services() # Plugs do phoenix https://hexdocs.pm/phoenix/Phoenix.Controller.html (o use, SuaAppWeb, :controller)
 |> action()          # lib/sua_app_web/controllers/* (sua função)
```

*Pipe das ações:

```elixir
connection
  |> find_user() # lib/sua_app/* (ainda não vimos isso)
  |> view()      # lib/sua_app_web/views/*
  |> template()  # lib/sua_app_web/templates/*
```

*imagem do post de [Marek Piwnicki](https://unsplash.com/@marekpiwnicki)*