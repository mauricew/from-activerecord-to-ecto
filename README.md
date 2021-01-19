From ActiveRecord to Ecto
====================
>This guide is in the early stages and goes mostly in depth into querying, with other sections being built out.

If you're evaluating the journey from Ruby/Rails over to an Elixir based stack such as Phoenix, this guide will certainly help in the database department. Several patterns used in the ActiveRecord ORM are not necessarily shared with Ecto (which additionally isn't considered an ORM).  
Much of what developers do with ActiveRecord make data models that function the same for all consumers (with the use of validations and callbacks), while Ecto encourages less strict behavior. While it may be possible to define these boundaries in ActiveRecord (think [dry-rb](https://dry-rb.org)), certain applications may not do this, which can make the transition a little difficult.  
Finally, Rails has very nice guides for hands-on use of ActiveRecord. Ecto has great API documentation but there's no procedural guides making it easy to translate over code. Detailed examples that would mimic real use is what I'll try to cover within these documents.

## Preface
This guide suggests a basic understanding of Elixir syntax and terminology, as well as data structures when to use a keyword list or a map.  
Not to mention an open mind - you'll want to step outside the active record mindset and go toward more domain driven design. Ecto promotes separation of business and data layer logic by not having mandatory callback methods or a single source of validations.

### Format
Examples will be described as follows:

````ruby
puts "First with a ruby example,"
````

````elixir
IO.puts "followed by an equivalent elixir example."
````
Some describing remarks may be put before or after each code block.

### Code jumpstart
To reduce the amount of typing necessary, whether in a code file or the console, there are a few things you can do:
* Import the required modules, e.g. `Ecto.Query` and `Ecto.Changeset`
* Alias your application's `Repo` module
* Alias the types you will be using

~~~~elixir
import Ecto.{Changeset, Query}
alias MyApp.Repo
alias MyApp.Accounts.User
~~~~

## Contents
* [Schemas](schemas.md)
* [Validations](validations.md)
* [Querying](querying.md)
* [Data changes](data-changes.md)
* [Callbacks](callbacks.md)


## Sections to add to this guide
* Error handling
* Transactions
