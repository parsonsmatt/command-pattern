# Final Boss

Note:

Okay, so that's the easiest and most common form of the command pattern.
We can use our easy to test, easy to understand, and easy to read functions to
generate a list of actions to take, and then have dead simple handlers to
interpret the commands into real actions.

There's something missing, though.


# Changing mid course?

Note:

What if we want to change our commands based on a return value of a handler?
Let's look at the following code.


```ruby
def charge_user(user)
  user.subscriptions.each do |subscription|
    if user.balance? >= subscription.price
      user.charge!(subscription.price)
    else
      send_balance_notice!(user, subscription)
    end
  end
end
```

Note:

I've used the question mark to indicate input effects, and the exclamation mark
to indicate the output effect. We iterate over the users subscriptions, and if
the user's current balance is at least the subscription price, then we charge
the subscription to their account.

We can't just take the balance as a parameter, because it changes during execution.


```ruby
def subs_to_charge(user)
  balance = user.balance?
  result = []
  subscriptions.each do |subscription|
    if balance >= subscription.price
      result << subscription
      balance -= subscription.price
    end
  end
  result
end
```

Note:

We could create an array of subscriptions to charge for based on the users
current balance, and then run charge! on those subscriptions.


```ruby
def charge_user(user)
  charges = subs_to_charge(user)
  charges.each do |subscription|
    user.charge!(subscription.price)
  end

  # Array difference:
  subs_to_notify = user.subscriptions - charges
  subs_to_notify.each do |subscription|
    send_balance_notice!(user, subscription)
  end
end
```

Note:

This is about as nice as that looks. It's still not great. Let's use the command pattern.


## Identify Commands

```ruby
Charge =
  Struct.new(:user, :amount)

SendBalanceNotice = 
  Struct.new(:user, :subscription)

UserBalance = 
  Struct.new(:user)
```

Note:

Our three commands are to Charge the user, send a balance notification, and *query* the users balance.
The query class is new. It represents a *query* that we want to make. It's like
an input effect, but it is supposed to change throughout the execution of the plan.

After identifying commands, we need an interpreter. The interpreter is pretty
easy to create, even with this new restriction.

Functionally speaking, the interpreter is mapping over the structure of our
computation. They're "maps". We need a way to construct our action plans
dynamically, without having access to the actual values.


## Allowing input effects...

```ruby
def charge_user(user)
  user.subscriptions.map do |subscription|
    if user.balance? >= subscription.price
      Charge.new(user, subscription.price)
    else
      SendBalanceNotice.new(user, subscription)
    end
  end
end
```

Note:

If we allow input effects, then we just build the command list as normal. This
can be an OK solution if your input queries aren't too complicated or difficult to deal with. However, we *really* don't want to have to deal with that. So let's figure out how to *build* our dynamic instructions without actually running the effects.


# What to do next?

```ruby
[ action_1, action_2, action_3 ]
```

Note:

With the array mapping, we answer "What do we do next" by taking the next item
in the array. If we want to have a more dynamic structure, then we can't be
using an array. Let's add an `:and_then` parameter to each command.


# Next!

```ruby
Charge = 
  Struct.new(:user, :amount, :and_then)
        #    ^      ^        ^
        #    |      |        +-- output
        #    +------+----------- input
```

Note:

So, this is a new Charge command that has an and_then parameter to specify what
happens afterwards. What is "next" going to look like? Well, Charge doesn't
have a meaningful output, so the next thing in the sequence won't take any
input. Since we're talking about the next action to take in our sequence, it
should be a command.


# And then, ...

```ruby
UserBalance =
  Struct.new(:user, :and_then)
        #    ^      ^
        #    |      +-- output
        #    +--------- input
```

Note:

This is our new User Balance struct. It has an `and_then` parameter also. This
parameter will get access to the balance returned from the command when it is
finally executed, and will use that information to construct the next action in
the sequence.


# Finally!

```ruby
Return = Struct.new(:result)
```

Note:

We need to have some way of terminating the chain of execution. This class does
it. We call it `Return` because we can think of it like the `return` keyword in
imperative programming: "stop executing and return this value".


# A single charge

Note:

Let's look at how this is with a single charge.


```ruby
def charge_or_email(user, subscription)
  UserBalance.new(user, -> (balance) do
    if balance >= subscription.price
      Charge.new(user, subscription.price, -> do
        Return.new(user) 
      end)
    else
      SendBalanceNotice.new(
        user, subscription, -> do
          Return.new(user) 
        end
      )
    end
  end)
end
```

Note:

So, we create our first command. We check the user balance. The first parameter
is the user, and the second parameter is a function that accepts the balance
and determines what to do next. If the balance is greater than or equal to the
price, then we create a Charge command on the user with that price, and finally
Return the user. Otherwise, we create a SendBalanceNotice command with the user
and the subscription.


# Interpreting

Note:

Interpreting these commands is relatively straight forward.


```ruby
class UserBalanceStripe
  def run(command)
    user = command.user
    balance = user.balance?
    command.and_then.call(balance)
  end
end
```

Note:

For the user balance, We extract the user from the command, call Stripe to get
the user's balance, and pass the balance to the command's "And then" method.
Finally, we return the command that is generated from this lambda.


```ruby
class UserBalanceLocal
  attr_reader :balances

  def initialize(balances)
    @balances = balances
  end

  def run(command)
    balance = balances[command.user.id]
    command.and_then(balance)
  end
end
```

Note:

We can also create a mock interpreter, that lives locally. Here we initialize
it with a mapping from user IDs to their balances, and when we receive the
command, we just check that hash map.


```ruby
class ChargeStripe
  def run(command)
    command.user.charge!(command.subscription.price)
    command.and_then.call
  end
end
```

Note:

For charging against stripe, it's basically the same thing. We call the charge method on the command's user, and then generate the next command in the sequence.


# Put it all together

Note:

Now, our functions are returning the next thing in the sequence. But we haven't put them all together yet.
Whereas the older command pattern used a "map" to go over the commands, we need to "reduce" over them.


# Reduce

```ruby
[1, 2, 3]                    #collection
  .reduce(0,                 # starting value
  -> (accumulator, next_value) do
    accumulator + next_value # reducer
  end
)
# => 6
```

Note:

Let's recap reducing, since it can be difficult. Reducing a collection takes a combination function, a starting value, and a collection of things to reduce. We take the first item in the collection, combine it with the starting value using our combination function. That gives us a new reducing value, which we then combine with the second element of our collection.


# Illustrated

Note:

Since reduce can be a bit tricky to get at first, here's how it evaluates.


```
f = -> (acc, x) { acc + x }

[1, 2, 3].reduce(0, f) 

[2, 3].reduce( f(0, 1), f)
[2, 3].reduce( 1, f)

[3].reduce(f(1, 2), f)
[3].reduce(3, f)

[].reduce(f(3, 3), f)
[].reduce(6, f)

6
``` 

Note:

We start by defining our combiner, which just adds the two numbers. We pluck
the first number off the array, pass it to the combining function with the
initial accumulator 0, and that becomes our new accumulator. We continue
plucking the first element off and combining with the accumulator until the
array is empty. Once the array is empty, the accumulator is returned.


# Reducing Command Trees

```ruby
class Interpeter
  attr_reader :handlers

  def initialize(handlers)
    @handlers = handlers
  end

  # ...
```

Note:

This can be a bit tricky to think about. Like the ComposeCommand class we made
earlier, we've got our hash of handlers that we're initialized with. 


```ruby
  # ...

  def interpret(command)
    case command
    when Return
      return command.result
    else
      handler = handlers[command.class]
      next_command = handler.run(command)
      self.interpret(next_command)
    end
  end
end
```

Note:

We have two cases: If the command is a Return, then we terminate execution and return whatever the result of the command is.
Otherwise, we get our handler for the given command, and run it with the command.
This gives us a next command to work with.
We then recurse on the function, calling `interpret` on the next command.


# Grafting Trees

Note:

There's one last piece of the puzzle. We need a way to compose two command trees. That is, given some command sequence, we need to have a way to say "And after you do all of this stuff, run this other sequence of commands."


```ruby
def charge_or_email(user, subscription)
  UserBalance.new(user, -> (balance) do
    if balance >= subscription.price
      Charge.new(user, subscription.price, -> do
        Return.new(user) 
      end)
    else
      SendBalanceNotice.new(
        user, subscription, -> do
          Return.new(user) 
        end
      )
    end
  end)
end
```

Note:

Here's our command structure for deciding whether to charge or email a user.
Now, we need some way to compose it, so we can iterate over all of the users
subscriptions and run this command tree.

We could naively generate a list of these commands and then run the interpreter
over each one individually.

Or we could bind the sequences together!


# Return = Terminate

Note:

We terminate execution when we use the Return command.
So we should be able to bind two trees together by replacing the Returns in the original tree with the sequences we're building out.


```ruby
def Interpreter
  attr_reader :binds
  def initialize
    @binds = []
  end

  def bind(next)
    @binds << next
  end

  # ...
```

Note:

First, we create an empty list of binds, and we expose a function for binding
another command to our interpreter. These binds are going to be the same thing
as the `and_then` parameters we've been dealing with: a function that returns
a command.


```ruby
  def interpret(command)
    case command
    when Return
      result   = command.result
      and_then = binds.shift

      if and_then
        interpret(and_then.call(result))
      else
        result
      end
    else 
      # ...
```

Note:

We want to identify the Returns, or ends, in our current command structure. If we have a return, then we want to take the first "next path" out of our array, and interpret that path. Otherwise, we just return the result of this and we're done.


```
def charge_user(user, subscriptions)
  interpreter = Interpreter.new
  user.subscriptions.each do |subscription|
    interpreter.bind -> {
      UserBalance.new user, -> (balance) {
        if balance >= subs.price
          Charge.new user, subs.price, -> {
            Return.new nil
          }
        else
          SendBalanceNotice.new user, subs, -> {
            Return.new nil
          }
        end
      }
    }
  end
end
```

Note:

So this is how we'd write that with our new command pattern.

The syntax is a little ugly. Let's write some helpers that clean it up.


```ruby
def charge(user, amount)
  Charge.new(user, amount, Return.new nil)
end

def send_balance_notice(user, subscription)
  SendBalanceNotice.new(user, sbscription, Return.new nil)
end

def user_balance(user)
  UserBalance.new(user, Return.new nil)
end
```

Note:

So these functions make up our "empty constructors." These trees just do one thing, and then they return nothing. 


```ruby
class Command
  def bind(and_then)
    old = @and_then
    @and_then = -> { and_then.call(old.call) }
  end
end

class UserBalance < Command
  def bind(and_then)
    old = @and_then
    @and_then = -> (balance) do
      and_then.call(old.call(balance))
    end
  end
end
```

Note:

We'll also add the following command on a now shared super class. So when we call `bind` on an individual command, it'll call the new bound function with the result of calling the old bound function. The default implementation just calls the new method with the result of the old method.

For classes that have meaningful return values, like UserBalance, the new lambda needs to take that balance, pass it to the old method, and then pass the result of that into the new method.


# Improved Syntax!

```ruby
def charge_user(user)
  user.subscriptions.reduce(Return.new nil) 
    do |commands, subscription|
    commands.bind -> do
      user_balance(user).bind -> (balance) do
        if balance >= subscription.price
          charge(user, subscription.price)
        else
          send_balance_notice(user, subscription)
        end
      end
    end
  end
end
```

Note:

So this syntax is a good bit nicer. It's not as clean as the plain ol' imperative syntax, but it gives us something much more powerful.

The return of this is a data structure. It's a series of instructions that we can carry out and interpreter however we like.

All we have to do is provide the right interpreters for each of the individual commands, and the code will glue and compose super nicely.


# Intrigued?

## It's a free monad

Note:

If you want to learn more about this final evolution of the command pattern,
the theory behind it is called the free monad. It's super interesting and very
powerful. Unfortunately, most typed languages can't express it well. Haskell
and Scala both have great free monad libraris, but Java/F#/C#/etc. are unable
to express these things.

Dynamic languages aren't restricted by types, but it's on you to ensure everything plugs together well.
