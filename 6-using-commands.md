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
fooResultInterpreter :: InsertFooResult -> DB ()
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


```haskell
let (value, action) = Foo.myFunc 1 2 3

insertFooResult     
  :: InsertFooResult -> DB ()

testInsertFooResult 
  :: InsertFooResult -> State MockDB ()

redisFooResult      
  :: InsertFooResult -> Redis ()
```


# Back to Billing Logic

Note:

OK, that's great, but let's get back to the important stuff -- getting money
from people!


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
Likewise, since they have the same duck type, we can pass any old instance of a
subscriber interpreter to a class.


```haskell
stripeSubscriber   
  :: SubscribeCommand -> IO ()
internalSubscriber 
  :: SubscribeCommand -> IO ()
testSubscriber     
  :: SubscribeCommand -> State User ()
```


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


# Functor

```haskell
executeCommands
  :: [SubscribeCommand]
  -> (SubscribeCommand -> IO a)
  -> [IO a]
executeCommands commands interpreter =
  map interpreter commands
```

Note:

Well, unfortunately `map` just doesn't quite handle it like we'd want to. In
Ruby, this would iterate over each item in the array, execute the callback, and
return the array of all results. In Haskell, what we get is a list of IO
operations -- nothing actually happens here, we just prepare the instructions
to be executed later. We're looking for something a little more powerful.


# Traversable

```haskell
executeCommands
  :: [SubscribeCommand]
  -> (SubscribeCommand -> IO a)
  -> IO [a]
executeCommands commands interpreter =
  traverse interpreter commands

executeCommands' 
  :: (Traversable t, Applicative f)
  => t SubscribeCommand
  -> (SubscribeCommand -> f a)
  -> f (t a)
executeCommands' = for
```

Note:

In Haskell, if you want to iterate over a collection along with some effect,
then you want to use Traversable. A Haskell traversal is like an effectful map.


# Dumb Data

## Is Easy <!-- .element: class="fragment" -->

Note:

Commands are just dumb data. Since they're dumb data, we can easily stuff them
in data structures are do interesting things to them. While the above example
just used an array, you could easily have a lazy list of commands, or a binary
tree, or a dictionary, or whatever.

In Ruby, we can rely on the Enumerator module. Haskell has the Traversable type
class.


# Optimizing Commands

Note:

Since we have introduced a data layer between business logic and execution, we
can easily optimize command sequences.
The additional data separation allows us to act with more knowledge about what
it is we're doing.


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

Here's our naive logic. It *works*. But it's inefficient! We issue three SQL
queries here, when we could save a tremendous amount of time by doing a bulk
insert.

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


```haskell
data FooResult 
  = InsertFooResult FooResult
  | BulkFooResult [FooResult]

isInsert :: FooResult -> Bool
isInsert x = case x of
  InsertFooResult _ -> True
  _ ->                 False

optimize :: [FooResult] -> [FooResult]
optimize commands = 
    rest ++ [BulkFooResult (map unwrap inserts)]
  where
    (inserts, rest) = partition isInsert commands
    unwrap (InsertFooResult x) = x
```

Note:

Here's the Haskell variant of the above Ruby code. We have to specify what the 
possibilities of the command are, but we can easily provide the optimizations.

This partial function kind of sucks, so don't actually use this in the real
life. I don't recommen this exact code structure, but I unfortunately have to
fit stuff on slides.
