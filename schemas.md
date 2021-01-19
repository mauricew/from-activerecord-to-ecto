## Schemas

### Associations

#### `belongs_to`
The belongs_to association is mostly the same.
Ecto doesn't infer the schema module so you need to define it yourself.

````ruby
class Address
  belongs_to: :user

  # The equivalent of belongs_to :user, class_name: 'User'
end
````

````elixir
schema "addresses" do
  belongs_to :user, User
end
````

##### Different foreign key on the same schema

````ruby
class Address
  belongs_to: :user, foreign_key: :account_id
end
````

````elixir
schema "addresses" do
  belongs_to :user, User, foreign_key: :account_id
end
````

##### Different foreign key on the referenced schema

````ruby
class Address
  belongs_to: :user, foreign_key: :account_id
end
````

````elixir
schema "addresses" do
  belongs_to :user, User, references: :account_id
end
````

##### Using a scope

````ruby
class Address
  belongs_to: :user, -> { where(activated: true) }
end
````

````elixir
schema "addresses" do
  belongs_to :user, User, where: [activated: true]
end
````