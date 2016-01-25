# JaResource

A behaviour to reduce boilerplate in your JSON-API compliant Phoenix
controllers with out sacrificing flexibility.

## Rational

JaResource lets you focus on the data in your APIs, instead of worrying about
response status, rendering validation errors, and inserting changesets.

** DISCLAIMER: This is curretly pre-release software **

## Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed as:

  1. Add ja_resource to your list of dependencies in `mix.exs`:

        def deps do
          [{:ja_resource, "~> 0.0.1"}]
        end

  2. Ensure ja_resource is started before your application:

        def application do
          [applications: [:ja_resource]]
        end

While not required, it is suggested to direct JaResource what repo to use in
controllers:

    config :ja_resource,
      repo: MyApp.Repo


## Usage

For the most simplistic resources JaSerializer lets you replace hundreds of
lines of boilerplate with a simple use statement. JaResource simply defines
the standard restful controller actions for you, while providing many simple
callbacks you can optionally implement to customize behaviour.

To expose index, show, update, create, and delete of the `MyApp.Post` model
with no retrictions:

```elixir
defmodule MyApp.V1.PostController do
  use MyApp.Web, :controller
  use JaResource
end
```

You can optional restrict JaResource to only implement the actions you don't
want to define yourself (however there are better ways to tweak behavior):

```elixir
defmodule MyApp.V1.PostsController do
  use MyApp.Web, :controller
  use JaResource, except: [:delete]

  def delete(conn, params) do
    # Custom delete logic
  end
end
```

And because JaResource is just implementing actions, you can still use plug
filters just like in normal Phoenix controllers:

```elixir
defmodule MyApp.V1.PostsController do
  use MyApp.Web, :controller
  use JaResource

  plug MyApp.Authenticate when action in [:create, :update, :delete]
end
```

You are also free to define any custom actions in your controller, JaResource
will not interfear with them at all.

```elixir
defmodule MyApp.V1.PostsController do
  use MyApp.Web, :controller
  use JaResource

  def publish(conn, params) do
   # Custom action logic
  end
end
```

### Changing the model exposed

By default JaResource parses the controller name to determine the model exposed
by the controller. `MyApp.UserController` will expose the `MyApp.User` model,
`MyApp.API.V1.CommentController' will expose the `MyApp.Comment` model.

This can easily be overriden by defining the `model/0` callback:

```elixir
defmodule MyApp.V1.PostsController do
  use MyApp.Web, :controller
  use JaResource

  def model, do: MyApp.Models.BlogPost
end
```

### Customizing records returned

Many applications need to expose only subsets of a resource to a given user,
those they have access too or maybe just models that are not soft deleted.
JaResource allows you to define the `records/1` and `record/2`

`records/1` is used by index, show, update, and delete requests to get the base
query of records. Many/most controllers will override this:

```elixir
defmodule MyApp.V1.MyPostController do
  use MyApp.Web, :controller
  use JaResource

  def model, do: MyApp.Post
  def records(%Plug.Conn{assigns: %{user_id: user_id}}) do
    model
    |> where([p], p.author_id == ^user_id)
  end
end
```

`record/2` receives the results of `records/1` and the id param and returns a
single record for use in show, update, and delete. This is less common to
customize but may be useful if using non-id fields in the url:

```elixir
defmodule MyApp.V1.PostController do
  use MyApp.Web, :controller
  use JaResource

  def record(query, slug_as_id) do
    query
    |> MyApp.Repo.get_by(slug: slug_as_id)
  end
end
```
