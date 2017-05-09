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
We can't really do that.
In fact, if we look at the type of our commands, we'll see that there's *no way* for a command to affect another command!


# Firewalled

```haskell
data SubscribeCommand
  = Subscribe User Plan

data List a = Nil | Cons a (List a)

type Commands = List SubscribeCommand
```

Note:

A Subscribe command has no way of returning a value or meaningfully affecting
the commands that comes next. We know this through parametricity -- the type of
a list doesn't allow the `a` parameters to influence the structure of the list.

The same problem exists for binary trees, streams, or whatever other
traversable data structure you can think of.

Lets look at some business logic that would be impossible to implement as this sort of command.


# Hmm...

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

We can't just take the balance as a parameter, because it changes during
execution when we charge the user. 

Let's start by identifying the commands we'd need to implement this.


## Identify Commands

```ruby
Charge =
  Value.new(:user, :amount)

SendBalanceNotice = 
  Value.new(:user, :subscription)

UserBalance = 
  Value.new(:user)
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


# What to do next?

```ruby
[ action_1, action_2, action_3 ]
```

Note:

With the array mapping, we answer "What do we do next" by taking the next item
in the array. If we want to have a more dynamic structure, then we can't be
using an array, or other traversable structure. Let's add an `:and_then`
parameter to each command.


# Next!

```ruby
Charge = 
  Value.new(:user, :amount, :and_then)
        #    ^      ^        ^
        #    |      |        +-- output
        #    +------+----------- input
```

Note:

So, this is a new Charge command that has an `and_then` parameter to specify what
happens afterwards. What is "next" going to look like? Well, Charge doesn't
have a meaningful output, so the next thing in the sequence won't take any
input. Since we're talking about the next action to take in our sequence, it
should be a command.


# Next!

```haskell
data ChargeCommand
  = Charge User Amount ChargeCommand
```

- Do this, and then do this next thing

Note:

I often find, even when programming in Ruby or PHP, that it's extremely helpful
to write data definitions in Haskell, especially if things are starting to get
complicated. This is a formulation of the previous Ruby command. It's a
recursively defined data type!

Now, let's cover the novel bit -- the input query, where we ask the user's
balance.


# And then, ...

```ruby
UserBalance =
  Value.new(:user, :and_then)
        #    ^      ^
        #    |      +-- output
        #    +--------- input
```

Note:

This is our new User Balance value. It has an `and_then` parameter also. This
parameter will get access to the balance returned from the command when it is
finally executed, and will use that information to construct the next action in
the sequence. This means it will be a callback!

Let's write that in Haskell:


# And then, ...

```haskell
data ChargeCommand
  = Charge User Amount ChargeCommand
  | UserBalance User (Amount -> ChargeCommand)
```

Note:

We've added a new command to our ChargeCommand data type. This one has a
*callback* from an Amount to a ChargeCommand when we construct it.


# Finally!

```ruby
Return = Value.new(:result)
```

Note:

We need to have some way of terminating the chain of execution. This command does
it. We call it `Return` because we can think of it like the `return` keyword in
imperative programming: "stop executing and return this value".


# Finally!

```haskell
data ChargeCmd
  = Return a
  | Charge User Amount (ChargeCmd a)
  | UserBalance User (Amount -> ChargeCmd a)
  | SendNotice User Subscription (ChargeCmd a)
```

(`Cmd` so it'll fit on the slide)

Note:

The Haskell code doesn't have anything to return with, really, so we make it an
empty constructor.

To keep things consistent in the Ruby, we'll make *all* of the `and_then`
parameters a callback. The no-value ones will just have empty callbacks, for
simplicity.


# A single charge

Note:

Let's look at how this is with a single charge.


<pre><code class="lang-ruby hljs" data-trim data-noescape>
pure = -> a { Return.new a }

def charge_or_email(user, subscription)
  UserBalance.new(user, -> (balance) do <span class="fragment">
    if balance >= subscription.price
      Charge.new(
        user, subscription.price, pure
      )</span><span class="fragment">
    else
      SendBalanceNotice.new(
        user, subscription, pure
      )
    end</span>
  end)
end
</code></pre>

Note:

So, we create our first command. We check the user balance. The first parameter
is the user, and the second parameter is a function that accepts the balance
and determines what to do next. If the balance is greater than or equal to the
price, then we create a Charge command on the user with that price, and finally
Return the user. Otherwise, we create a SendBalanceNotice command with the user
and the subscription.


```haskell
chargeOrEmail 
  :: User -> Subscription -> ChargeCommand
chargeOrEmail user sub =
  UserBalance user sub $ \balance -> 
    let price = subscriptionPrice sub in
    if balance >= subscriptionPrice sub
      then 
        Charge user price Return
      else 
        SendBalanceNotice user sub Return
```

Note:

This is the Haskell equivalent of the above code.
It's basically the same thing.


# Interpreting

Note:

Interpreting these commands is relatively straight forward.


## Interpreting 

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
Finally, we return the command that is generated from this lambda, which
generates the next command to interpret. 


## Interpreting 

```haskell
chargeCommand :: ChargeCommand -> IO ()
chargeCommand (UserBalance user andThen) = do
  balance <- Stripe.getBalanceFor user
  let nextCommand = andThen balance
  chargeCommand nextCommand
  
```

Note:

For handling this query, we get the user's balance from Stripe. We pass that to
`andThen`, which constructs the next command we need to interpret. We'll call
`chargeCommand` recursively to handle the next case.


## Locally Testing

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


## Locally Testing

```haskell
chargeLocally 
  :: ChargeCommand 
  -> State (Map UserId Amount) ()
chargeLocally (UserBalance user andThen) = do
  balance <- gets (Map.lookup (userId user))
  chargeLocally (andThen balance)
```

Note:

The Haskell is basically the same thing, just with different syntax.


```ruby
class ChargeStripe
  def run(command)
    price = command.subscription.price
    command.user.charge!(price)
    command.and_then.call(nil)
  end
end
```

Note:

For charging against stripe, it's basically the same thing. We call the charge
method on the command's user, and then generate the next command in the
sequence. We'll pass `nil` to that callback, sicne there's no meaningful value here.


# Put it all together

Note:

Now, our functions are returning the next thing in the sequence. But we haven't put them all together yet.
Whereas the older command pattern used a "map" to go over the commands, we need to "reduce" over them.


# Reduce

```ruby
[1, 2, 3]                      # collection
  .reduce(
    0,                         # starting value
    -> (accumulator, next_value) do
      accumulator + next_value # reducer
    end
)
# => 6
```

Note:

Let's recap reducing, since it can be a little tricky. Reducing a collection
takes a combination function, a starting value, and a collection of things to
reduce. We take the first item in the collection, combine it with the starting
value using our combination function. That gives us a new reducing value, which
we then combine with the second element of our collection.


## Reducing Command Trees

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
      Charge.new(user, subscription.price, -> _ do
        Return.new(user) 
      end)
    else
      SendBalanceNotice.new(
        user, subscription, -> _ do
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

We terminate execution when we use the Return command.  So we should be able to
bind two trees together by replacing the Returns in the original tree with the
sequences we're building out.


```ruby
def add_next(cmd, and_then)
  case cmd
  when Return
    and_then.call(cmd.result)
  when Charge
    Charge.new(
      cmd.user, cmd.price, -> () do
        add_next(cmd.and_then.call, and_then)    
      end
    )
  # ...
```

Note:

The basic structure that we'll be working with is to analyze the command, and
create a new copy of it with the same values. However, in that callback,
instead of calling it directly, we'll call it, and then call `add_next`
recursively with the callback. Eventually, we'll bottom out and hit the Return
class, where we'll pass the command result to it, thus generating the new
command.


```ruby
  when UserBalance
    UserBalance.new(
      cmd.user, -> (balance) do
        add_next(
          cmd.and_then.call(balance), and_then
        )
      end
    )
  # ...
end
```

Note:

The query uses a similar pattern. We create a new UserBalance command, and in
the callback, we pass the balance to the original command's callback, and
recurse into `add_next`.

These methods should really be on the class or module for these value objects:
specifically, we should have a way to map a function over the `and_then` part
of the value, keeping the rest the same, and the `add_next` method should also
be a class member.


keep it classy

```ruby
class UserBalance < Value(:user, :and_then)
  def add_next(&callback)
    self.with(and_then: (balance) -> do
      self.and_then
          .call(balance)
          .add_next(callback)
    end)
  end
```

Note:

OK, so this is how we'd extend the command classes to provide the `add_next`
method, and also the `map` method. And, we actually don't need any of the
details of the original class here -- we can abstract this out to The Scary 'M'
Word: module!


```ruby
module Command
  def add_next(&callback)
    self.with(and_then: (value) -> do
      self.and_then
        .call(value)
        .add_next(callback)
    end)
  end
end
```

Note:

Now, we can simply include this module in all of our commands, and we get the
add_next and map functions for free.


```ruby
class Charge < Value(:user, :amount, :and_then)
  include Command
end

class UserBalance < Value(:user, :and_then)
  include Command
end
```

Note:

So here's how we define our command types. We'll also want to have some helpers to make defining these things more convenient.


```ruby
pure = -> (x) do
  Return(x)
end

def user_balance(user)
  UserBalance.new(user, pure)
end

def charge(user, amount)
  Charge.new(user, amount, pure)
end

def send_balance_notice(user, subscription)
  SendBalanceNotice.new(user, subscription, pure)
end
```

Note:

The basic command terminates with a Return, so we'll terminate our singleton commands with this pure function.
The other helpers just make a new object of the right command class, and start it off with the Return constructor.
When we go to implement more of this stuff, we'll use the `add_next` method to build the chain.


```ruby
def charge_user(user)
  start = user_balance(user)

  user.subscriptions.reduce start do |cmd, sub|
    cmd.add_next do |balance|
      if balance >= sub.price
        charge(user, sub.price)
      else
        send_balance_notice(user, sub)
      end
    end.add_next do 
      user_balance(user) 
    end
  end
end
```

Note:

This is the logic now! We start off with checking the user balance. Then we
reduce over the subscriptions, and for each one, we extend the command, either
charging the user or sending a notification email.  We extend that command by
checking the user's balance.

This structure perfectly encodes the logic we want, and it won't be evaluated
until we actually run it. So we've successfully created an execution plan.

The syntax is a little nicer now -- you're pretty close to something
idiomatically Ruby. I bet we can get nice syntax in Haskell, too.


```haskell
data ChargeCmd a
  = Charge User Amount (ChargeCmd a)
  | UserBalance User (Amount -> ChargeCmd a)
  | SendBalanceNotice User Sub (ChargeCmd a)
  | Return a
```

Note:

Some of the nicer DSLs in Haskell use `do` syntax. If we weant `do`, then we
need to convert this data type into a Monad.  We've got a type parameter, we
just need to write the instance.  It turns out, the `add_next` function we
defined is exactly the code we need for our monad instance.


```haskell
instance Monad ChargeCmd where
  (>>=) :: ChargeCmd a 
        -> (a -> ChargeCmd b) 
        -> ChargeCmd b
  command >>= k =
    case command of
      Return a ->
        k a
      Charge user amount next ->
        Charge user amount (next >>= k)
      UserBalance user next ->
        UserBalance user (\balance -> 
          next balance >>= k)
      SendBalanceNotice user sub next ->
        SendBalanceNotice user sub (next >>= k)
```

Note:

So here's our monad instance. The weird symbol function up there is pronounced
"bind", and it takes a ChargeCmd as it's first argument. The second argument is
a function which accepts an "a" value, returning a ChargeCmd b. The return is a
ChargeCmd b.

We recursively dig deeper and deeper into the command structure, until we
eventually find a 'Return' constructor. Then we apply the callback to the value
contained there.  Let's look at our program in Haskell now, to see how we can
take advantage of the new syntax.


## The Helpers

```haskell
userBalance :: User -> ChargeCmd Double
userBalance user = UserBalance user Return

charge :: User -> Amount -> ChargeCmd ()
charge user amount = 
  Charge user amount (Return ())

sendBalanceNotice 
  :: User -> Subscription -> ChargeCmd ()
sendBalanceNotice user subscription =
  SendBalanceNotice user subscription (Return ())
```

Note:

These guys are going to help our syntax look nice, by presenting a way of lifting our command data structure into a more ordinary function call, easily compatible with the do notation.


# YEAAHHHH

```haskell
chargeSubscriptions :: User -> ChargeCmd ()
chargeSubscriptions user =
  for_ (userSubscriptions user) $ \sub -> do
    balance <- userBalance user
    let price = subscriptionPrice sub
    if balance >= price
      then charge user price
      else sendBalanceNotice user sub
```

Note:

Oh yeah!!! This is awesome. We're using entirely normal Haskell syntax to
express our business logic, without having to do anything too weird or fancy. 
Indeed, this looks almost exactly like the code we'd run in IO, but instead of
running directly in IO, we're just constructing a data structure.


# Intrigued?

## It's a free monad

Note:

If you want to learn more about this final evolution of the command pattern,
the theory behind it is called the free monad. It's super interesting and very
powerful. Unfortunately, most typed languages can't express it well. Haskell
and Scala both have great free monad libraries, but Java/F#/C#/etc. are unable
to express these things.

Dynamic languages aren't restricted by types, but it's on you to ensure
everything plugs together well.
