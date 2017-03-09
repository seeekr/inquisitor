# Inquisitor

Easily build composable queries for Ecto.

[![Build Status](https://secure.travis-ci.org/DockYard/inquisitor.svg?branch=master)](http://travis-ci.org/DockYard/inquisitor)

## Usage

Adding Inquisitor to a project is simple:

```elixir
defmodule MyApp.PostController do
  use Inquisitor

  def index(conn, params) do
    posts =
      App.Post
      |> build_query(params)
      |> Repo.all()

    json(conn, posts)
  end
end
```

After `use Inquisitor` a `build_query/2` is added to
the `MyApp.PostController`. It takes a queryable variable and the
params as arguments.

This sets up a key/value queryable API for the `Post` model. Any
combination of fields on the model can be queried against. For example,
requesting `[GET] /posts?foo=bar&baz=qux` will create the query:

```sql
SELECT p0."foo", p0."baz" FROM posts as p0 WHERE (p0."foo" = $1) AND (p0."baz" = $1);
```

`$1` and `$2` will get the values of `"bar"` and `"qux"`,

### Security

By default Inquisitor is an opt-in library. It will not provide any
querying access to any key/value pair. The params list will be iterated
over and a no-op function is called on each element. You must add custom
query handlers that have a higher matching order on a case by case
basis.

If you'd like to add a catch-all for any key/value pair you can override
the default:

```elixir
defquery key, value do
  query
  |> Ecto.where([r], field(r, ^String.to_existing_atom(attr)) == ^value)
end
```

However, this is not recommended.

### Adding custom query handlers

You can use the `defquery` macro to create custom query handlers:

```elixir
defmodule MyApp.PostsController do
  use Inquisitor

  def index(conn, params) do
    posts =
      App.Post
      |> build_query(params)
      |> Repo.all()

    json(conn, posts)
  end

  defquery "inserted_at, date do
    query
    |> Ecto.Query.where([p], p.inserted_at >= ^date)
  end
end
```

### Handing fields that don't exist on the model

The keys you query against don't need to exist on the model. Revisting
the date example, let's say we want to find all posts inserted for a
given month and year:

```elixir
defquery attr, value when attr == "month" or atr == "year" do
  query
  |> Ecto.Query.where([e], fragment("date_part(?, ?) = ?", ^attr, e.inserted_at, type(^value, :integer)))
end
```

`defquery` can use guards, this should help you easily build solutions
for complex queries.

### Usage Outside of Phoenix Controllers

To use inside a module other than a Phoenix Controller, you'll need to import `Ecto.from/1` otherwise you may see an error like `cannot use ^value outside of match clauses`.

Note: we use `warn: false` to suppress an incorrect warning generated by Elixir thinking `from` is unused.

```elixir
defmodule MyApp.PlainModule do
  import Ecto.Query, only: [from: 1], warn: false
  use Inquisitor
end
```

## Authors

* [Brian Cardarella](http://twitter.com/bcardarella)

[We are very thankful for the many contributors](https://github.com/dockyard/inquisitor/graphs/contributors)

## Versioning

This library follows [Semantic Versioning](http://semver.org)

## Want to help?

Please do! We are always looking to improve this library. Please see our
[Contribution Guidelines](https://github.com/dockyard/inquisitor/blob/master/CONTRIBUTING.md)
on how to properly submit issues and pull requests.

## Legal

[DockYard](http://dockyard.com/), Inc. &copy; 2016

[@dockyard](http://twitter.com/dockyard)

[Licensed under the MIT license](http://www.opensource.org/licenses/mit-license.php)
