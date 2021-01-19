## Querying

### Basic rules
In ActiveRecord, your query is executing directly using the model's class name. Several class (static) methods are used to access different query

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

The execution of queries are done with just a few functions that are called on the Repo.  
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
User.find!(1) # raises ActiveRecord::RecordNotFound
~~~~

Ecto:
~~~~elixir
User |> Repo.get(1)
User |> Repo.get!(1) # raises Ecto.NoResultsError
~~~~

With a non-PK field:

ActiveRecord:
~~~~ruby
User.find_by(first_name: "John")
User.find_by!(first_name: "John")
~~~~

Ecto:
~~~~elixir
User |> Repo.get_by(first_name: "John")
User |> Repo.get_by!(first_name: "John")
~~~~

Now let's assume there are two users with the name "John".
* **ActiveRecord** does a `LIMIT 1` query and does not care about multiple records with the same value.  
* **Ecto** does no limit and throws an error if there are multiple records with the same value.

This is a good way to clean up some hidden undesirable practices that AR does.

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

Now that queries are getting a little more wordy, it's time to go off in one of two directions for Ecto:

* Functional: Keep piping functions in. This can be done with a keyword list for single table queries or an expression for more complex ones.
* Declarative: The use of a LINQ-style DSL.

~~~~elixir
# The functional method using a keyword map syntax,
# best with simple single table queries
User
|> where(first_name: "John")
|> Repo.all

# The functional method using an expression
User
|> where([u], u.first_name == "John")
|> Repo.all

# The declarative method
query = from u in User,
        where: u.first_name == "John"
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
Equivalent to `WHERE first_name IN ('John', 'Erin')`
~~~~ruby
User.where(first_name: ["John", "Erin"])
User.where.not(first_name: ["John", "Erin"])
~~~~

~~~~elixir
User 
|> where([u], u.name in ["John", "Erin"])
|> Repo.all

User 
|> where([u], not (u.name in ["John", "Erin"]))
|> Repo.all
~~~~

#### Comparison operators
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
~~~~ruby
# A high level of verbosity
User.where(first_name: "John")
    .or(User.where(first_name: "Jack"))
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

Note that in Rails, `limit` is the correct way to run the query within the database. `take` would not allow any additional querying logic to be added and is unoptimized (since the data is materialized into an array before that point).

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
|> Repo.preload([addresses: order(Address, :state)])
~~~~

### Grouping
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
AR autoincludes the grouped field in its query. With Ecto you'll need to define the selected colums here.
Ecto doesn't have a built in `COUNT(*)` but you can use the PK.

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
