## Data changes
_this section under construction_

It's time to learn about the changeset, the key structure used across data inserts and updates around Ecto.

### The changeset
In its most basic form it acts like a hash, but it can go way beyond that.  
Let's start with something basic. As seen in above method calls, ruby can take in keyword args (or you can explicitly pass as a hash).

~~~~ruby
User.create(first_name: "John", last_name: "Douglass")
~~~~

Changesets look like elixir maps with the addition of the type name in the prefix.
~~~~elixir
new_user = %User{first_name: "John", last_name: "Douglass"}
new_user |> Repo.insert
~~~~
