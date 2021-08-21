---
layout: post
title: Programando phoenix - Cap 3 - Controllers
date: 2021-08-21 08:00:00 -0300
image: phoenix.jpg
tags: Funcional, Elixir, Phoenix
---

## Disclaimer
- Os posts dessa série são minhas anotações pessoais dos meus estudos do livro "Aprendendo programação funcional com elixir.
- Eles não passaram por revisão, possuem typos e alguns erros.
- Estão mais próximos de pensamentos desconexos do que de um texto estruturado.
- Dito isso, seguem anotações (:

# Capitulo 03 - Controllers
* Vimos no capítulo passado como funciona toda a pipeline do phoenix
* Fizemos uma aplicação Hello world pra ver por cima como as coisas funcionam, e vamos agora entender a fundo como funcionam os controllers e todo o ecossistema de contexto, views e layouts

## Entendendo controllers
* Nesse capitulo vamos começar a fazer uma aplicação do 0, e nós vamos adicionando coisas durante o restante do livro.
* A idéia é fazer uma plataforma que você pode inserir um link de um video presente em outra plataforma, como youtube por exemplo, e as pessoas podem comentar. Os comentários são atualizados em tempo real e pode ser feito por várias pessoas diferentes.
* Nesse capitulo vamos começar pelos controllers
* Inicialmente vamos focar em um controller que lide com os usuários. 
* Só pra relembrar um pouco do conteudo do [[02. The lay of the land|Capítulo passado]], o pipeline de uma requisição é o seguinte:
```elixir
 connection
 |> endpoint()              # lib/sua_app_web/endpoint.ex
 |> router()                # lib/sua_app_web/router.ex
 |> browser_pipeline()      # (também é acessível no arquivo de rota)
 |> UserController.action() # lib/sua_app_web/controllers/user_controller.ex
 
 ###### DESTRINCHANDO O CONTROLLER ######
 
 connection
 |> UserController.index()
 |> UserView.render("index.html")
 
```
* Quando chamamos a função `index` no nosso controller, ela vai buscar todos os usuários, mas essa ação pode criar uma série de perguntas relacionadas a lógica de negócio. Onde estão armazenados os dados desse usuário? Que informações do usuário nós vamos exibir? E se os usuários tiverem diferentes papeis, vamos mostra todos? As informações são todas públiucas?
* O controller não é o melhor lugar para escrever lógica de negócio. Fica mais interessante isolar essas questões em contextos.

### Contextos
* No final do dia um contexto nada mais é do que um módulo que agrupa função que possui propositos similares.
* Como exemplo, nossa aplicação precisa ler, modificar e deletar contas do usuário, então é interessante deixar todas elas agrupadas no mesmo canto.
* De forma geral, podemos dizer que um contexto em phoenix encapsula toda a lógica de negócio que possui um propósito comum.
* Separando toda a lógica de negócio em uma camada, conseguimos evitar duplicação do código, e conseguimos centralizar código com o mesmo propósito em um único lugar.
*  E no final das contas, precisamos lidar com algo que interaja com as regras de negócio, e é pra isso que os controllers existem.
*  Nos esforçamos para deixar que os contextos não saibam da existencia dos controllers, e os controllers não saibam de regra de negócio. 
*  Mas por que fazemos isso? Mantenabilidade. Conseguimos fazer várias modificações de regras de negócios semprecisar encostar na camada web da nossa aplicação. Da mesma forma, conseguimos testar a nossa lógica de negócio sem precisar tocar em testes de integração.

### Criando o projeto
* Vamos criar um projeto novo chamado `rumbl`
* no terminal digitamos `mix phx.new rumbl`
* Quando perguntarem se é pra instalar as dependencias, a gente digita `y` (de `yes`)
* Nesse capitulo não vamos interagir com banco de dados, porém a partir do próximo vamos começar a brincar com ecto, então pra facilitar a vida do nosso eu do futuro, aproveitaremos o momento pra criar nosso banco de dados
* Criamos o banco com o comanto `mix ecto.create`
* Agora que está tudo pronto, seria interessante ver se tudo esta funcionando. Digitamos `mix phx.server` no terminal, abrimos o navegador e vamos navegar até `http://localhost:4000`
* Se não deu erro e apareceu algo com a logo do phoenix, deu bom \o/

### Uma homepage simples
* A página que apareceu é um template que tem um html. Ela pode ser encontrada no seguinte arquivo: `lib/rumbl_web/templates/page/index.html.eex`
* Podemos fazer algumas modificações no html dela pra aparecer o que quisermos. 
* Se você fizer as modificações e for no navegador, graças ao hot reload, as suas modificações já devem estar disponiveis :D 
* Agora que já temos uma página, podemos por em prática o que falamos acima. No momento não queremos integrar com um banco de dados, mas podemos criar uns usuários falsos, juntamente com algumas funções que interagem com os dados do usuário.
* O primeiro passo é criar um contexto

### Trabalhando com contextos
* Contexto nada mais é do que a API das regras de negócio da sua aplicação, não API web, mas a interface pública da sua aplicação, ou seja, todas as funções públicas que você está expondo para que a sua camada web possa acessar.
* Usando contexto, nós conseguimos desacoplar o seu sistema em partes independentes, que fica muito mais tranquilo de lidar. 
* Com isso em mente vamos criar uns usuários de exemplo, chumbando eles na nossa lógica de negócio. Os usuários estarão chumbados e serão substituidos posteriormente por usuários presentes em um banco de dados, mas a nossa inferface pública continuará a mesma.
* Se a gente for lembrar um pouco a nossa lógica de pastas, dentro da pasta `lib` nós temos duas pastas: `rumbl` e `rumbl_web`. Com base no que falamos sobre separar a nossa lógica de negócio da camada web, faz sentido que os usuários fiquem na pasta `rumbl`.
* O primeiro passo é pensar onde a nossa funcionalidade deveria estar. Contas de usuários é um conceito fundamental pra a nossa aplicação, então podemos criar os nossos usuários dentro de um módulo chamado `Accounts`. No futuro podemos extender esse módulo para todas as funções relacionadas a contas, como autenticação e reset de senha.
* Vamos criar um arquivo chamado `lib/rumbl/accounts/user.ex` e vamos criar uma estrutura de dados representando usuários (que em elixir chamamos de struct)
```elixir
defmodule Rumbl.Accounts.User do ## Convenção de nomenclatura
 defstruct [:id, :name, :username] ## <--- aqui, ó
end
```
* Com o código acima criamos uma struct `Rumbl.Accounts.User` que contém os campos `id`, `name` e `username`. 
* Structs em elixir é a nossa principal abstraçãop pra trabalhar com dados estruturados

### Structs em elixir
* Structs em elixir são contruidas em cima de maps e nada mais é do que um Map com algumas propriedades extras. 
* Aqui são as principais diferenças:
  * É mais fácil para previnir erros de digitação, já que uma struct possui campos pré-definidos
  * Conseguimos preencher com valores default
  * A verificação de maps só acontecem em runtime (ouseja, depois que você compilou sua aplicação e resolve rodar ela), enquanto strucs são verificados em compile time (se tiver errado, você não consegue nem compilar o programa, muito menos rodar)
```elixir
iex> alias Rumbl.Accounts.User # (vai facilitar escrever os structs)

##### Criando um usuário com um typo usando maps
iex> usuario = %{usrmname: "rachelcurioso"}
%{usrmname: "rachelcurioso"}
iex> usuario.username
***** erro :P  (ele vai dizer que não achou a chave "username")

##### Criando um usuário com typo usando structs
iex> usuario = %User{nme: "Rachel Curioso"}
*********** Erro (ele vai dizer que não achou a chave "nme")

##### Criando um usuário sem typo com structs , preenchendo só um campo
iex> usuario = %User{name: "Rachel Curioso"}
%Rumbl.Accounts.User{id: nil, name: "Rachel Curioso", username: nil}
```

Com esse conhecimento em mente, vamos criar nossos usuários nesse arquivo `lib/rumbl/accounts.ex`

```elixir
defmodule Rumbl.Accounts do
  @moduledoc """
  The Accounts context
  """
  
  alias Rumbl.Accounts.User
  
  def list_user do
    [
      %User{id: 1, name: "Elaine Watanabe", username: "elainew"}
      %User{id: 2, name: "Juliana Helena", username: "julianah"}
      %User{id: 1, name: "Rachel Curioso", username: "rachelc"}
    ]
  end
 
 def get_user(id) do
  Enum.find(list_users(), fn map -> map.id == id end)
 end
 
 def get_user_by(params) do
  Enum.find(list_users(), fn map ->
    Enum.all?(params, fn {key, val} -> Map.get(map, key) == val end)
  end)
 end
end
```

* No módulo acima definimos a interface pública do nosso contexto de `Accounts`. e já conseguimos brincar com ela no terminal.
* Agora que já temos a nossa lógica de negócio, conseguimos criar nosso controller para que nossa aplicação web consiga interagir com a regra de negócio.

## Construindo um controller.
* Já criamos um controller no capitulo anterior, e agora precisamos repetir a façanha :D 
* No phoenix conseguimos criar tudo apartir de um único comando no terminal, porém como estamos conhecendo phoenix é melhor criar tudo na mão primeiro pra saber como tudo funciona.
* O primeiro passo é ir criarmos a rota para definirmos qual o endpoint que o usuário vai acessar, qual o controller que deve ser procurado, e qual a ação que precisamos tomar.
* Vamos abrir nosso arquivo rota, localizar o pedaço de código abaixo e adicionarmos nossa rota
```elixir
scope "/" RumblWeb do
  pipe_through :browser
 
  get "/", PageController, :index
end

################ vai ficar assim:

```elixir
scope "/" RumblWeb do
  pipe_through :browser
 
  get "/users", UserController, :index      #<---- novo
  get "/users/:id", UserController, :show   #<---- novo
  get "/", PageController, :index
end
```
* Agora adicionamos a rota, apontamos para o controller que iremos criar, e pra ação que queremos.
* O nome `:index` e `:show` é uma convenção de aplicações web. A ação `:index` é usada quando vamos listar todos os itens de uma determinada entidade, enquanto `:show` é usado quando vamos mostrar os detalhes de uma única entidade.
* Além de `:index` e `:show` também temos `:new`, `:create`, `:update`, `:edit` e `:delete` (geralmente new é a página que cria novas entidades, o create é a ação de salvar uma nova entidade no banco, o edit a página de editar uma entidade, o update é a ação de salvar edições no banco, e deletar é a ação de apagar uma entidade)
* Agora vamos analisar uma das linhas que nós adicionamos:
* `get "/users", UserController, :index`
* A primeira palavra é o `get`, que significa que vamos fazer uma requisição http do tipo GET (se fosse uma ação para criar um novo usuário, seria um POST, por exemplo).
* A segunda palavra é a rota que nós queremos que o usuario acesse.
* A terceira é o controller que o usuário vai encontrar as ação que ele procura (no caso listar todos os usuários)
* A quarte é a ação que vai ser executada quando ele for no controller. Ela é o mesmo nome da função que vai ser executada.
* OKEI
* agora que criamos a rota, já temos acesso a ela através do nosso navegador através do endereço `http://localhost:4000/users` (só vai dar um erro, porque não temos controller nem view ainda, mas a rota ta lá! Não vai dar um 404, por exemplo).
* Agora que temos a rota, precisamos criar um controller (vou numerar alguns pontos importantes para falar sobre eles abaixo)
```elixir
defmodule RumblWeb.UserController do
  use RumblWeb, :controller   # *1
  
  alias Rumbl.Accounts  # *2
  
  def index(conn, _params) do
    users = Accounts.list_users()    # *2
    render(conn, "index.html", users: users)    # *3
  end
end
```
  1. Precisamos especificar que isso é um controller para conseguirmos usar as funções que o phoenix disponibilizam para nós, como `render`, por exemplo
  2. Para listarmos todos os usuários, nós precisamos acessar a função que lista os usuários, que criamos na interface da nossa lógica de negócio, que é o módulo `Rumbl.Accounts.list_users()` (usamos um alias pra ficar mais legível). 
  3. Agora que temos todos os usuários, podemos passar para as nossas views como um dos argumentos da função render. Dessa forma, todos os usuários vão estar acessíveis para nós na view através da variável `@users` (ps: se nós tivessimos passado `pastel: users` a variável que teriamos acesso na view seria `@pastel`)
 
 ### Codando as views
 * Os termos views e templates geralmente são uma coisa só em alguns outros frameworks, mas em phoenix há uma separação para tornar tudo mais explicito. 
 * As views estão concentrados funções que convertem dados que o usuário final vai consumir, enquanto templates são fragmentos de página web.
 * Criaremos uma view na pasta `lib/rumbl_web/views/user_view.ex` e adicionamos o seguinte código
```elixir
defmodule RumblWeb.UserView do
  use RumblView, :view    # *1
  
  alias Rumbl.Accounts    # *3
  
  def first_name(%Accounts.User{name: name}) do   # 2* e 3*
    name
    |> String.split(" ")
    |> Enum.at(0)
  end
end
```
1. Primeiro nós definimos que isso é uma view, assim nosso módulo vai ter acesso as funções de views que o phoenix fornece e consiga converter os templates html em algo que o phoenix lide melhor.
2. Criamos uma função auxiliar que identifique qual o primeiro nome do usuário
3. Pra isso precisamos ter acesso ao struct que nós criamos na nossa camada de negócio, então definimos o alias do nosso contexto, e acessamos o dado "name" do struct que sabemos que virá

* OKEI! Agora é hora de criar nosso template `.html.eex`
* vamos na pasta de templates e criamos o arquivo `lib/rumbl_web/templates/index.html.eex`
```html
<h1> Todos os usuários </h1>

<table>
  <%= for user <- @users do %>    # 1*
    <tr>
      <td><b><%= first_name(user) %></b> (<%= user.id %>)</td>    # 2*
      <td><%= link "+", to: Routes.user_path(@conn, :show, user.id) %></td>    # 3* e 4*
    </tr>
  <% end %>
</table>
```
* Antes de explicar o que é cada um desses elementos, é interessante explicar o que é o `<%= =>` (e o `<% %>`, que não está no código mas tem um papel similar)
* quando encontramos esse simbolo no nosso arquivo `html.eex`, sabemos que estamos lidando com função elixir. Quando o código começa com `<%=`, significa que o que está dentro dele será renderizado na tela, já o `<%` também é código elixir, mas o seu conteudo não é renderizado na tela. Geralmente só colocamos na view os códigos que precisam ser renderizados na tela, os que não precisam ficam no controle (poderiamos fazer um `<% users = Accounts.list_users() %>` na nossa view, porém deixar esse tipo de código na view complica a mantenabilidade do seu código.
* Beleza, agora explicando o que é cada um desses elementos
1. Nessa linha nós estamos fazendo um loop na lista de `@users` que definimos no controller. Para cada usuário `user` dessa lista (que é um mapa), será desenhada uma linha na tabela com duas colunas. A primeira com o nome, e a segunda com o link para a página de show dele.
2. Aqui nós usamos o `<%=` para renderizar o nome e o id do usuário que está passando pelo loop
3. O link é um helper que as views passam pro template. Falaremos dele em breve (mas vale frizar agora que ele gera uma tag html cujo texto é `+` e o caminho linkado é a rota show do usuário)
4. Quando o link é para uma rota da nossa aplicação, nós podemos usar essa sintaxe `Routes.algo_path(@conn, :acao, parametro`). Para saber as rotas disponíveis, podemos digitar no terminal `mix phx.routes`.
* No livro o Chris nos explica que templates em phoenix são rápidos porque são uma lista. Em outras linguagens o template é uma grande string

## Usando helpers
* Elixir tem alguns vários helpers. Link é um deles
* Você consegue também consegue ter acesso a essas funções no seu terminal iterativo.
```bash
iex> Phoenix.HTML.Link.link("Home", to: "/")
{:safe, [60, "a", [[ (....)]} 
```
* O retorno vai ser uma tupla, cujo primeiro valor é `:safe`, seguido de uma sequencia de valores.
* O phoenix tem proteção automatica contra ataques chamado de injeção de script através de html, por isso que o primeiro valor da tupla é "safe", o que significa que não temos scripts escondidos no nosso link.
* Se convertermos o resultado do nosso link para html `Phoenix.HTML.Link.link("Home", to: "/") |> Phoenix.HTML.safe_to_string()` o resultado vai ser uma tag link html `<a href="\"/\">Home</a>` cujo o texto do link vai ser o primeiro argumento que passamos na função `link`, que leva para o endereço que definimos no segundo argumento.
* Se você tiver se perguntando de onde vem essas funções de helpers, elas vem da view. Quando usamos o `use RumblWeb, :view`, nós estamos dando acesso para aquele módulo acessar todas as funções presentes no arquivo `lib/rumbl_web.ex`. 
* Se formos nesse arquivo, vemos uma boa quantide de código disponível para uso. Sempre podemos colocar nossos códigos lá também porém, vale lembrar que esse código será importado para todas as suas views e templates, então é interessante deixar ele o mais enxuto possível.

* Okei, dada essa explicação, agora podemos ir no nosso navegador e ver os 3 usuários que nós criamos!
* Criamos um link para exibir o usuário nos nossos templates, na nossa rota, porém faltam algumas coisas: Colocar a ação no controller, criar a view (e dar uma enxugada em código repetido)

## Mostrando um usuário

* Se formos na rota, podemos ver a segunda função que nós criamos hoje, a `get "/user/:id", UserController, :show`
* Se olharmos para o endereço dessa rota, vemos que temos um `:id`. Quando aparece um atom na nossa rota, estamos falando que aquilo é uma variável. Temos acessoa essa variável do nosso controller, e podemos usá-la em funções :D 
* Essa variável está junto com a `conn`, mas graças a um dos plugs que nós usamos nessa aplicação, essa variável vem separadinha e pronta pra uso no `params`.
* Podemos abrir o `UserController` que criamos e adicionar a nossa ação de `show`
```elixir
def show(conn, %{"id" => id}) do
  user = Accounts.get_user(id)
  render(conn, "show.html", user: user)
end
```
* Asssim como no index, nós pegamos o id que chegou para nós, passamos para uma função que criamos no módulo `Accounts`, buscamos um usuário e levamos ele para a view através da função `render`.
* Agora criamos um arquivo chamado `lib/rumbl_web/user/show.html.eex` e adicionamos o seguinte código
```html
<h1>Mostrando o usuário:</h1>
<b><%= first_name(@user) %></b> (<%= @user.id %>)</b>
```
* Se você for no navegador e apontar para `/user/1` vai ver os dados sobre nosso usuário (:

### Convenção de nomes
* Quando o phoenix renderiza um template de um controller, ele infere o nome da view a partir do seu nome. Então um controller chamado `UserController` vai procurar por uma view chamada `UserView`. Se seu controle fosse `PastelController`, ele procuraria por um módulo de view chamado `PastelView`.
* Em alguns frameworks, você precisa usar plural em alguns casos e singular em outros. No phoenix não tem disso, embora você possa configurar caso queira.

### Aninhando templates
* Dito isso, podemos voltar a olhar um pouco pros templates. 
* se você reparar os templates de `show` e `index` vai encontrar essa linha em comum: `<b><%= first_name(@user) %></b> (<%= @user.id %>)</b>`
* Quando temos código que se repete em mais de um template, podemos abstrair a parte repetida para outro arquivo e reutilizá-la depois :D 
* Podemos criar um arquivo chamado `user.html`,  por esse pedaço de código lá e, chamar esse novo template no nosso arquivo `show` e `index`.
* O show ficaria assim:
```html
<h1>Mostrando o usuário:</h1>
<%= render "user.html", user: @user %> ## (precisamos lembrar de passar as variáveis usadas
```
* o index ficaria assim:
```html
<h1> Todos os usuários </h1>

<table>
  <%= for user <- @users do %>
    <tr>
      <td><%= render "user.html", user: user %></td>
      <td><%= link "+", to: Routes.user_path(@conn, :show, user.id) %></td>   
    </tr>
  <% end %>
</table>
```
* Vale lembrar que as views são só uma função, e que se quisermos acessá-la através do terminal, nós conseguimos também.
* Podemos também chamar a função render direto da view, sem precisar criar um arquivo pra cada template. Podemos ver isso no nosso módulo `RumblWeb.ErrorView`, no qual listamos nossas páginas de erro

### Layouts
* Falamos de views, templates e agora é a vez dos layouts :D 
* Os nomes são parecidos, então depois de explicar sobre layouts a gente faz uma colinha
* Sabe quando você abre um site, ele tem um header, o corpo e o footer? O corpo muda quando você vai clicando nos menus, mas o header e o footer continum lá.
* Isso acontece porque o header e o footer são um Layout, e lá são injetados as views/templates
* Se você for em `lib/rumbl_web/templates/layout/app.html.eex` nós vamos achar esse pedaço de código
```html
<main role="main" class="container">
  <p class="alert alert-info" role="alert">
    <%= get_flash(@conn, :info) %>
  </p>
  <p class="alert alert-danger" role="alert">
    <%= get_flash(@conn, :error) %>
  </p>
  <%= render @view_module, @view_template, assigns %>
</main>
```
* É aqui que suas views e templates são adicionadas na tela. 
* A ordem de renderização do phoenix é Layout > View > Template
* Você pode editar o template para ficar mais ao seu gosto (: Lembra só que ele vai ser o mesmo todas as páginas.
* E é isso :D O próximo capitulo vamos entender melhor o que é ecto e o que ele come

*imagem do post de [Marek Piwnicki](https://unsplash.com/@marekpiwnicki)*