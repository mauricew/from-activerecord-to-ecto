## Callbacks considered harmful (and what to use instead)
ActiveRecord has many built-in callbacks that allow you to perform actions before or after model creation, deletion, update, and more. You can about them [here](http://guides.rubyonrails.org/active_record_callbacks.html).  
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
  |> put_change(:username_lower, changeset.username |> String.downcase)
end
~~~~

Remember to use this changeset (or a higher order changeset that uses it) whenever making changes to a user.
