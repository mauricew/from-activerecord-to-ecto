## Validations

Default validations are mostly the same across the two.

### Built-in validations

#### Requiring a field

````ruby
class User
  validates :name, presence: true
  # or
  validates_presence_of :name
end
````

````elixir
def changeset(struct, params) do
  struct
  |> cast(params, [:name])
  |> validate_required(:name)
end
````

NOTE: ActiveRecord's definition of "required" includes a rule that a boolean field would need to be `true`. In Ecto, `true` or `false` passes this check.

#### Inclusion/Exclusion

````ruby
validates :name,
  inclusion: { in: %(John Jack) },
  exclusion: { in: ["Jub Jub"] }
````

````elixir
struct
|> cast(params, [:name])
|> validate_inclusion(:name, ["John", "Jack"])
|> validate_exclusion(:name, ["Jub Jub"])
````

#### Length

````ruby
validates :country, length: { is: 2 }
validates :password, length: { min: 8, max: 32 }
# or
validates :password, length: { in: 8..32 }
````

````elixir
struct
|> cast(params, [:country, :password])
|> validate_length(:country, is: 2)
|> validate_length(:password, min: 8, max: 32)
````

#### Acceptance

````ruby
validates :finna_woke, acceptance: true
````

````elixir
struct
|> cast(params, [:finna_woke])
|> validate_acceptance(:finna_woke)
````

#### Confirmation

````ruby
  validates :email, confirmation: true
````

````elixir
struct
|> cast(params, [:email])
|> validate_confirmation(:email)
````

#### Numerics

````ruby
  validates :age, numericality: { only_integer: true, greater_than_or_equal_to: 18 }
````

````elixir
struct
|> cast(params, [:age])
|> validate_numericality(:age, greater_than_or_equal_to: 18)
````

NOTE: Ecto does not required an `only_integer` validation since the schema definition of an integer will make this check for any floating point or decimal number.

### Custom validations
With ActiveRecord, custom validations are created by subclassing `ActiveModel::Validator`.

````ruby
class MyValidator < ActiveModel::Validator
  def validate(record) do
    # Logic goes here
  end
end
````

````elixir
defp do_something(changeset) do
  # Logic goes here
end
````
