From ActiveRecord to Ecto
====================
>This guide is in the early stages and goes mostly in depth into querying, with other sections being built out.

If you're evaluating the journey from Ruby/Rails over to an Elixir based stack such as Phoenix, this guide will certainly help in the database department. Several patterns used in the ActiveRecord ORM are not necessarily shared with Ecto (which additionally isn't considered an ORM).  
Much of what developers do with ActiveRecord make data models that function the same for all consumers (with the use of validations and callbacks), while Ecto encourages less strict behavior. While it may be possible to define these boundaries in ActiveRecord (think [dry-rb](https://dry-rb.org)), certain applications may not do this, which can make the transition a little difficult.  
Finally, Rails has very nice guides to use ActiveRecord. Ecto has good API documentation but there's no procedural guides making it easy to translate over code. Detailed examples that would mimic real use is what I'll try to cover with this document.

## Preface
This guide suggests a basic understanding of Elixir syntax and terminology, as well as data structures when to use a keyword list or a map.  
Not to mention an open mind - you'll want to step outside the active record mindset and go toward more domain driven design. Ecto promotes separation of business and data layer logic by not having mandatory callback methods or a single source of validations.

### Code jumpstart
To reduce the amount of typing necessary, whether in a code file or the console, there are a few things you can do:
* Import the `Ecto.Query` and `Ecto.Changeset` modules
* Alias your application's `Repo` module
* Alias the types you will be using

~~~~elixir
import Ecto.{Query, Changeset}
alias MyApp.Repo
alias MyApp.Accounts.User
~~~~

## Querying

### Basic rules
In ActiveRecord, your query is executing directly using the model's class name.

~~~~ruby
# To get all records
User.all
~~~~

With Ecto, the `Repo` object is your gateway to the database.
~~~~elixir
# Call a function directly
Repo.all(User)
# Or with use the pipe operator
User |> Repo.all
~~~~

The execution of queries are done within a few functions that are called on the Repo.  
Here are the most common ones.
~~~~elixir
# To return multiple records
Repo.all
# To return a single record, and raise an error if more than one record is returned
Repo.one
# To check for existence, new for 3.0
Repo.exists?
~~~~

### Finding a record
#### Unique finders
In this situation the primary key is an integer with the value **1**.

ActiveRecord:
~~~~ruby
User.find(1)
~~~~

Ecto:
~~~~elxir
User |> Repo.get(1)
~~~~

With a non-PK field:

ActiveRecord:
~~~~ruby
User.find_by(first_name: "John")
# To throw exception if not found:
User.find_by!(first_name: "John")
~~~~

Ecto:
~~~~elixir
User |> Repo.get_by(first_name: "John")
User |> Repo.get_by!(first_name: "John")
~~~~

Now let's assume there are two users with the name "John".
ActiveRecord does a `LIMIT 1` query and does not care about multiple records with the same value, but Ecto does no limit and throws an error if there are multiple records with the same value.  
This is a good way to clean up some bad practices that AR does.

~~~~elixir
# This raises Ecto.MultipleResultsError if there's more than one record
User |> Repo.get_by(first_name: "John")
~~~~

#### Ordinal finders
With no order by to go from, it will sort by the primary key ascending or descending, and take the first/last object.

ActiveRecord:
~~~~ruby
User.first
User.last
~~~~

Ecto:
~~~~elixir
User |> first |> Repo.one
User |> last |> Repo.one
~~~~

### Filtering

Here it is in ActiveRecord:
~~~~ruby
User.where(first_name: "John")
~~~~

Now that queries are getting a little more wordy, it's time to go off in one of two directions:

* Functional: Keep piping functions in. This can be done with a keyword list for single table queries or an expression for more complex ones.
* Declarative: The use of a LINQ-style DSL.

~~~~elixir
# The functional method using a keyword map syntax
User
|> where(first_name: "John")
|> Repo.all

# The functional method using an expression
User
|> where([u], u.first_name == "John")
|> Repo.all

# The declarative method, useful for related entities
# since each get their own variable
query = from u in User,
        where: u.first_name == "John",
        select: [u]
query |> Repo.all
~~~~

#### Negation
~~~~ruby
User.where.not(first_name: "John")
~~~~

~~~~elixir
User
|> where([u], not (u.first_name == "John"))
|> Repo.all
# You may prefer this instead
User
|> where([u], u.first_name != "John")
|> Repo.all
~~~~

#### Multiple values
Equivalent to `WHERE first_name IN ('John', 'Maggie')`
~~~~ruby
User.where(first_name: ["John", "Maggie"])
User.where.not(first_name: ["John", "Maggie"])
~~~~

~~~~elixir
User 
|> where([u], u.name in ["John", "Maggie"])
|> Repo.all

User 
|> where([u], not (u.name in ["John", "Maggie"]))
|> Repo.all
~~~~

#### Inequality logic
ActiveRecord has two schools of thought here:
* Parameterized strings
* Arel
~~~~ruby
# You either did this
User.where('score >= ?', 100)
# Or this
uarel = User.arel_table
User.where(uarel[:score].gteq(100))
~~~~
but it's a lot more pleasant looking on the other side.
~~~~elixir
User
|> where([u], u.score >= 100)
|> Repo.all
~~~~

#### OR statements
For one-offs, Ecto makes things less verbose. Anything more complicated and you gotta repeat the full query.

~~~~ruby
# A high level of verbosity
User.where(first_name: "John").or(User.where(first_name: "Jack"))
~~~~

~~~~elixir
User
|> where(first_name: "John")
|> or_where(first_name: "Jack")
|> Repo.all
~~~~

### Sorting
When specifying a descending sort, the syntax is reversed from ActiveRecord's.

~~~~ruby
User.order(:first_name)
User.order(first_name: :desc)
~~~~

~~~~elixir
User
|> order_by(:first_name)
|> Repo.all
User
|> order_by(desc: :first_name)
|> Repo.all
~~~~

How about multiple columns?
~~~~ruby
User.order(:last_name, :first_name, age: :desc)
~~~~

~~~~elixir
User
|> order_by([:last_name, :first_name, desc: :age])
|> Repo.all
~~~~


### Pagination (Offset/Limit)
They're essentially the same.
~~~~ruby
User.order(:first_name).limit(5).offset(10)
~~~~

~~~~elixir
User
|> order_by(:first_name)
|> limit(5)
|> offset(10)
|> Repo.all
~~~~

Note that in Rails, `limit` is the correct way to run the query whereas `take` would not allow any additional querying logic to be added  (since the data is materialized into an array at that point).

### Loading related data
ActiveRecord has a method called `includes` that runs either `preload` or `eager_load` depending on some magic.
Ecto just has `preload`, which does a separate query for the related data.

~~~~ruby
User.preload(:addresses).find(1)
~~~~

~~~~elixir
User 
|> Repo.get(1)
|> Repo.preload([:addresses])
~~~~

A big improvement is the ability to customize the inner query by adding filtering or sorting with a preload. With AR, you either need your `has_many` to use a custom scope, or do an `eager_load`.

~~~~elixir
User
|> Repo.get(1)
|> Repo.preload([addresses: Address |> order(:state)])
~~~~

### Grouping
Pretty straightforward stuff.
~~~~ruby
User.group_by(:first_name).select(:first_name)
~~~~

~~~~elixir
User
|> group_by(:first_name)
|> select([:first_name])
|> Repo.all
~~~~

#### Aggregates
AR autoincludes the grouped field in its query.  
Ecto doesn't let you do `COUNT(*)` but you can use the PK.

~~~~ruby
User.count

User.group(:first_name).count
# Returns { 'John' => 1 } etc.
~~~~

~~~~elixir
User
|> Repo.aggregate(:count, :id)

# Just the count
User
|> Repo.aggregate(:count, :first_name)

# With the field grouped on
User
|> group_by(:first_name)
|> select([u], [u.first_name, count(u.id)])
~~~~

## Manipulation
### Inserts
It's time to learn about the changeset. In its most basic form it acts like a hash, but it can go way beyond that.

Let's start with something basic. As seen in above method calls, ruby can take in keyword args (or you can explicitly pass as a hash).

~~~~ruby
User.create(first_name: "John", last_name: "Douglass")
~~~~

Changesets look like elixir maps with the addition of the type name in the prefix.
~~~~elixir
new_user = %User{first_name: "John", last_name: "Douglass"}
new_user |> Repo.insert
~~~~

## Callbacks (or lack thereof)
ActiveRecord has many built-in callbacks that allow you to perform actions before or after model creation, deletion, update, and more. Read about them [here](http://guides.rubyonrails.org/active_record_callbacks.html).  
Ecto once had this concept, but it was removed in version 2.0, citing [the lack of fine-grained control they create](http://blog.plataformatec.com.br/2015/12/ecto-v1-1-released-and-ecto-v2-0-plans/). Of course, the replacement is to make a more complex changeset.

Let's take this example AR model, inspired by [Discourse](https://github.com/discourse/discourse), that converts a user's username to lowercase and stores it in a separate field.

~~~~ruby
# user.rb

validates_presence_of :username

before_save :update_username_lower

def update_username_lower
  self.username_lower = username.downcase
end
~~~~

So how does this translate? We do want to mimic the functionality of it always running, and it's as simple as piping an additional function into the changeset. (More functions in functional programming, what has the world come to?)  
Make sure the function returns the changeset as well.

~~~~elixir
# user.ex

def changeset(user, attrs)
  user
  |> cast(attrs, [:username])
  |> validate_required([:username])
  |> update_username_lower()
end

defp update_username_lower(changeset)
  changeset
  |> put_change(:username, changeset.username |> String.downcase)
end
~~~~

Remember to use this changeset (or a higher order changeset that uses it) whenever making changes to a user.

# Sections to add to this guide
* Error handling
* Data updates
* Validations
* Schemas
* Transactions
