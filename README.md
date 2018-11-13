# Demo of Elixir, Phoenix, GraphQL, Vue.js

Credit: https://codeburst.io/how-to-setup-graphql-vue-js-and-phoenix-1-3-part-1-the-backend-e3305641e5c

Create:

```sh
mix phx.new demo --no-brunch
```

Edit `./mix.deps` to add Absinthe, Poison, and change Cowbow to be a plug:

```elixir
defp deps do
[
  {:phoenix, "~> 1.3.0"},
  {:phoenix_pubsub, "~> 1.0"},
  {:phoenix_ecto, "~> 3.2"},
  {:postgrex, ">= 0.0.0"},
  {:phoenix_html, "~> 2.10"},
  {:phoenix_live_reload, "~> 1.0", only: :dev},
  {:gettext, "~> 0.11"},
  {:plug_cowboy, "~> 1.0"},
  {:absinthe, "~> 1.4.0"},
  {:absinthe_plug, "~> 1.4"},
  {:poison, "~> 3.1.0"}
]
end
```

Run:

```sh
mix deps.get
```

Edit `./lib/demo_web/router.ex` to append:

```elixir
forward "/graphql",
      Absinthe.Plug,
      schema: DemoWeb.Schema

forward "/graphiql",
      Absinthe.Plug.GraphiQL,
      schema: DemoWeb.Schema,
      interface: :simple
```

Edit `./lib/demo_web/endpoint.ex` to add the Absinthe plug parser:

```elixir
parsers: [:urlencoded, :multipart, :json, Absinthe.Plug.Parser],
```

Create `./lib/demo_web/schema.ex`:

```elixir
defmodule DemoWeb.Schema do
  use Absinthe.Schema
  import_types Absinthe.Type.Custom
  import_types DemoWeb.Schema.AccountTypes
  alias DemoWeb.Resolvers

  query do

    @desc "Get a user"
    field :user, :user do
      arg :id, non_null(:id)
      resolve &Resolvers.Accounts.find_user/3
    end

  end

end
```

Create a folder for resolvers:

```sh
mkdir ./lib/demo_web/resolvers
```

Create a file `./lib/demo_web/resolvers/accounts.ex`:

```elixir
defmodule DemoWeb.Resolvers.Accounts do 
  def find_user(_parent, %{id: id}, _resolution) do
    case Demo.Accounts.find_user(id) do
      nil ->
        {:error, "User ID #{id} not found"}
      user ->
        {:ok, user}
    end
  end
end
```

Create a file `./lib/demo/accounts.ex`:

```elixir
defmodule Demo.Accounts do
  # Stubbed out for now.
  def find_user(id) do
    %{
      name: "Alice Adams",
      id: id
    }
  end
end
```

Create a folder for web schema types:

```sh
mkdir ./lib/demo_web/schema
```

Create a file `./lib/demo_web/schema/account_types.ex`:

```elixir
defmodule DemoWeb.Schema.AccountTypes do
  use Absinthe.Schema.Notation

  @desc "A user"
  object :user do
    field :id, :id # clients can get the user id
    field :name, :string # clients can also ask for the name field
  end
end
```

Edit file `./lib/demo_web/templates/layout/app.html.eex` to change the body tag to:

```html
<body>
  <%= render @view_module, @view_template, assigns %>
  <script src="<%= static_path(@conn, "/js/app.js") %>"></script>
</body>
```

Edit file `./lib/demo_web/templates/page/index.html.eex` to replace the entire file with this:

```html
<div id=”app”></div>
```

Run this to create the database:

```sh
mix ecto.create
```

Run this to launch the server:

```sh
iex -S mix phx.server
```

Browse http://localhost:4000/graphiql

Try a query such as:

```graphql
{
  user(id: 1) {
    name
  }
}
```

Result:

```json
{
  "data": {
    "user": {
      "name": "Alice Adams"
    }
  }
}
```

