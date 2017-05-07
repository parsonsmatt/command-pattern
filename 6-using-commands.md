# Using Commands

Note:

Now that we've got all these commands, how do we actually use them? There are
a number of ways we can do this, in increasing complexity/power. As with most
things in software development, it's best to ask for as little power as you need.
Shooting yourself in the foot hurts a lot less if you only have a water gun.


# Command Interpreter

```ruby
class FooResultInterpreter
  def call(command)
    FooResult.insert(
      command.x, command.y, command.z
    )
  end
end
```

Note:

For each command, you implement an interpreter. Here's a basic interpreter for the FooResult class.


# Command Interpreter

```haskell
fooResultInterpreter :: InsertFooResult -> IO ()
fooResultInterpreter (InsertFooResult x y z) = 
   insert (FooResult x y z) 
```

Note:

Command interpreters in Haskell are just functions which convert a *command value* into an action of some sort.


# Single Command

```ruby
value, action = Foo.new.my_func(1,2,3)

InsertFooResultInterpreter.new.call(action)
# or,
TestInsertFooResultInterpreter.new.call(action)
# or,
RedisFooResult.new.call(action)
```

Note:

So, if you've got a single command, this is the easiest. You take that object
and you pass it to the interpreter of your choice. You can easily define many
different varieties of interpreters for a given command.


```ruby
class StripeSubscriber
  def call(command)
    Stripe::Subscription.create(
      command.user, command.plan
    ) 
  end
end

class InternalSubscriber
  def call(command)
    InternalBilling.create_subscription(
      command.user, command.plan
    )
  end
end
```

Note:

These classes share the same interface and can be used interchangeably.
So we can easily pass a single command to them and have it be executed.


# Running Multiple Commands

Note:

That covers the single command case, like our toy example. What about multiple
commands, like our more involved refactoring example? Since we're dealing with
simple values, we can use simple functions to chain commands. We'll start with
a simple case: multiple of the same command.


```ruby

commands = [
  Subscribe.new(x, y), 
  Subscribe.new(z, a),
  Subscribe.new(b, c)
]

# answer?
# wat do
# halp
```

Note:

Ok, so we've got an array full of Subscribe commands. And we want to actually execute each one.
Does anyone have a suggestion on how to do this?


```ruby
interpreter = StripeSubscriber.new
commands = [
  Subscribe.new(x, y), 
  Subscribe.new(z, a),
  Subscribe.new(b, c)
]

commands.map do |command| 
  interpreter.call command
end
```

Note:

We can simply iterate over the commands, and for each command, call it through
the interpreter. 


# Dumb Data

Note:

Commands are just dumb data. Since they're dumb data, we can easily stuff them
in data structures are do interesting things to them. While the above example
just used an array, you could easily have a lazy list of commands, or a binary
stree, or a hash, or whatever.


# Optimizing Commands

Note:

Since we have introduced a data layer between business logic and execution, we can easily optimize command sequences.


# InsertFooResult

```ruby
commands = [
  InsertFooResult.new(1, 2, 3),
  InsertFooResult.new(4, 5, 6),
  InsertFooResult.new(7, 8, 9)
]

commands.map { |cmd| interpreter.call cmd }
```

Note:

Here's our naive logic. It *works*. But it's inefficient! We issue three SQL queries here, when we could save a tremendous amount of time by doing a bulk insert.

Let's optimize this.


```ruby
def optimize(commands)
  inserts, rest = commands.partition do |command|
    command === InsertFooResult
  end

  rest.push(BulkInsert.new(inserts))
end
```

Note:

Ok, so here we're going to partition the commands into two lists; the first is
one where the command's class is an InsertFooResult. The second list is all of
the other commands. We return the list of non-insert commands with a BulkInsert
command appended to the end.
