# Arbor

[![Build Status](https://travis-ci.org/coryodaniel/arbor.svg)](https://travis-ci.org/coryodaniel/arbor)

Ecto nested-set and tree traversal using CTEs. Arbor uses a `parent_id` field
and CTEs to create simple deep nested SQL hierarchies.

This is currently a WIP. Not all tree methods have been implemented, see TODOs below.

## Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed as:

  1. Add `arbor` to your list of dependencies in `mix.exs`:

    ```elixir
    def deps do
      [{:arbor, "~> 0.1.0"}]
    end
    ```

  2. Ensure `arbor` is started before your application:

    ```elixir
    def application do
      [applications: [:arbor]]
    end
    ```

## Usage

```elixir
defmodule Comment do
  use Ecto.Schema
  # See config options below
  use Arbor.Tree, foreign_key_type: :binary_id

  schema "comments" do
    field :body, :string
    belongs_to :parent, Arbor.Comment

    timestamps
  end  
end
```

All methods return composable Ecto queries. For in depth examples see the [tests](./test/arbor)

### Roots

Returns root level records.

```elixir
roots = Comment.roots
        |> Repo.all
```


### Siblings

Return the siblings of a record.

```elixir
siblings = my_comment
           |> Comment.siblings
           |> Comment.order_by_popularity
           |> Repo.all
```

### Descendants

Returns the entire descendant tree of a record.

```elixir
descendants = my_comment
              |> Comment.descendants
              |> Comment.order_by_inserted_at
              |> Repo.all              
```

### Children

Returns the immediate children of a record.

```elixir
children = my_comment
           |> Comment.children
           |> Repo.all
```

### Parent

Returns the record's parent.

```elixir
parent = my_comment
         |> Comment.parent
         |> Repo.first
```

## Options

* *table_name* - set the table name to use in [CTEs](https://www.postgresql.org/docs/9.1/static/queries-with.html)
* *tree_name* - set the name of the CTE
* *primary_key* - defaults to field from Ecto's `@primary_key`
* *primary_key_type* - defaults to type from Ecto's `@primary_key`
* *foreign_key* - defauts to `:parent_id`
* *foreign_key_type* - defaults to type from Ecto's `@primary_key`
* *orphan_strategy* - defaults to `:nothing` currently unimplemented

## Contributing

You'll need PostgreSQL installed and a user that can create and drop databases.

You can specify it with the environment variable `ARBOR_DB_USER`.

The `mix test` task will drop and create the database for each run.

```elixir
mix test
```

## TODO

* Common tree operations
  * [x] descendants
  * [x] children
  * [x] siblings
  * [x] roots
  * [x] parent
  * [ ] ancestors
  * [ ] path (ancestors + target)
  * [ ] subtree (target + children)
  * [ ] delete (nilify, destroy, adopt)
  * [ ] move
  * [ ] leafs  
* [ ] Docs
* [ ] depth selector on descendants and subtree, include depth or array size in CTE?
* [ ] arbor.gen.migration SCHEMA_NAME FK_FIELD FK_TYPE
* [ ] Move each query/operation into its own module to make Arbor.Tree less busy.
* [ ] ID finders (?): root_ids, child_ids(target), etc...
* [ ] Boolean functions: is_root?, has_children?, etc...
* [ ] Support for multiple trees? The tree name could be prefixed on tree methods...
    ```elixir
      use Arbor, my_tree: opts, my_other_tree: more_opts
      ```
