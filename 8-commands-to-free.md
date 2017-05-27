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

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
data SubscribeCommand
  = Subscribe User Plan

data List a = Nil | Cons a (List a)

type Commands = List SubscribeCommand
</code></pre></div>

Note:

A Subscribe command has no way of returning a value or meaningfully affecting the commands that comes next.
We know this through parametricity -- the type of a list doesn't allow the `a` parameters to influence the structure of the list.

The same problem exists for binary trees, streams, or whatever other traversable data structure you can think of.

Lets look at some business logic that would be impossible to implement as this sort of command.


# Hmm...

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
def charge_user(user)
  <span class="fragment">user.subscriptions.each do |subscription|
    <span class="fragment">if user.balance? >= subscription.price
      user.charge!(subscription.price)</span>
    <span class="fragment">else
      send_balance_notice!(user, subscription)
    end</span>
  end</span>
end
</code></pre></div>

Note:

Alright, here's our code bit: we need to charge a user for all their subscriptions, unless they don't have enough cash, at which point we whine at them.

First, we iterate over the users subscriptions.
Then, if their account balance is at least the price of the subscription, we charge them for it.
I'm using the question mark to indicate input effects, and the exclamation mark to indicate the output effect here -- just to make it easy to see.
otherwise, we send a billing notice for the subscription that failed to charge.

We can't just take the balance as a parameter, because it changes during execution when we charge the user. 
So our old tricks won't work -- we actually need to account for this at runtime, by dynamically generating our commands somehow.

Let's start by identifying the commands we'd need to implement this.


## Identify Commands

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
Charge =
  Value.new(:user, :amount)

SendBalanceNotice = 
  Value.new(:user, :subscription)

UserBalance = 
  Value.new(:user)
</code></pre></div>

Note:

Our three commands are to Charge the user, send a balance notification, and *query* the users balance.
The query class is new.
It represents a *query* that we want to make.
It's like an input effect, but the information it returns is supposed to be made available to the rest of the plan.

This raises a question: How can our plans know what to do next?


# What to do next?

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
[ action_1, action_2, action_3 ]
</code></pre></div>

Note:

With the array mapping, we answer "What do we do next" by taking the next item in the array.
If we want to have a more dynamic structure, then we can't be using an array, or other traversable structure.
Let's add a `:do_next` parameter to each command.


# Next!

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
Charge = 
  Value.new(:user, :amount, :do_next)
        #    ^      ^        ^
        #    |      |        +-- output
        #    +------+----------- input
</code></pre></div>

Note:

So, this is a new Charge command that has a `next` parameter to specify what happens afterwards.
What is "next" going to look like? Well, Charge doesn't have a meaningful output, so the next thing in the sequence won't take any input.
Since we're talking about the next action to take in our sequence, it should be a command.  


# Next!

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
data ChargeCommand
  = Charge User Amount ChargeCommand
</code></pre></div>

- Do this, and then do this next thing

Note:

I often find, even when programming in Ruby or PHP, that it's extremely helpful to write data definitions in Haskell, especially if things are starting to get complicated.
This is a formulation of the previous Ruby command.
It's a recursively defined data type!

Now, let's cover the novel bit -- the input query, where we ask the user's balance.


# And then, ...

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
UserBalance =
  Value.new(:user, :do_next)
        #    ^      ^
        #    |      +-- output
        #    +--------- input
</code></pre></div>

Note:

This is our new User Balance value.
It has an `and_then` parameter also.
This parameter will get access to the balance returned from the command when it is finally executed, and will use that information to construct the next action in the sequence.
This means it will be a callback!

Let's write that in Haskell, and allow the types to guide our thinking.


# And then, ...

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
data ChargeCommand
  = Charge User Amount ChargeCommand
  | UserBalance User (Amount -> ChargeCommand)
</code></pre></div>

Note:

We've added a new command to our ChargeCommand data type. This one has a
*callback* from an Amount to a ChargeCommand when we construct it.


# Finally!

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
Return = Value.new(:result)
</code></pre></div>

Note:

We need to have some way of terminating the chain of execution.
This command does it.
We call it `Return` because we can think of it like the `return` keyword in imperative programming: "stop executing and return this value".
It just takes a single parameter and wraps it up, without providing a way to go forward.


# Finally!

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
data ChargeCmd a
  = Return a
  | Charge User Amount (ChargeCmd a)
  | UserBalance User (Amount -> ChargeCmd a)
  | SendNotice User Subscription (ChargeCmd a)
</code></pre></div>

(`Cmd` so it'll fit on the slide)

Note:

Since the Ruby `Return` can take a value of any type, we'll make the ChargeCommand type polymorphic in what it returns.
The command type has to be polymorphic now.
We can read the type of this as "A sequence of commands eventually returning an `a`".


# A single charge

Note:

Let's look at how this is with a single charge.
We won't do anything fancy -- we're just trying to get our basic stuff together for now.


<pre><code class="lang-ruby hljs" data-trim data-noescape>
def charge_or_email(user, subscription)
  UserBalance.new(user, -> (balance) do <span class="fragment">
    if balance >= subscription.price
      Charge.new(
        user, subscription.price, -> x { Return.new(x) }
      )</span><span class="fragment">
    else
      SendBalanceNotice.new(
        user, subscription, -> x { Return.new(x) }
      )
    end</span>
  end)
end
</code></pre>

Note:

So, we create our first command.
We check the user balance.
The first parameter is the user, and the second parameter is a function that accepts the balance and determines what to do next.
If the balance is greater than or equal to the price, then we create a Charge command on the user with the subscription's price.
We need to provide a callback returning a command for this, so we just Return whatever comes next.
Otherwise, we create a SendBalanceNotice command with the user and the subscription.  


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
chargeOrEmail 
  :: User -> Subscription -> ChargeCommand ()
chargeOrEmail user sub =
  UserBalance user sub $ \balance -> 
    let price = subscriptionPrice sub in
    if balance >= price
      then 
        Charge user price (Return ())
      else 
        SendBalanceNotice user sub (Return ())
</code></pre></div>

Note:

This is the Haskell equivalent of the above code.
It's basically the same thing.


# Interpreting

Note:

Interpreting these commands is relatively straight forward.
I'm going to do Haskell first, because the types with callbacks always gets a little tricky, and it's helpful for me to have the types to guide me.
Once I've found the right path, it's easy to erase the types and write it in Ruby.


## Interpreting 

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
data ChargeCommand a
  = UserBalance User (Amount -> ChargeCommand a)
  | ...

chargeCommand :: ChargeCommand a -> IO a
chargeCommand (UserBalance user balanceToCommand) = do
  balance <- Stripe.getBalanceFor user
  let nextCommand = balanceToCommand balance
  chargeCommand nextCommand
  
</code></pre></div>

Note:

Let's recall the type of the UserBalance constructor.
Our first argument is the user to get the balance for.
The second argument is a *function*, which accepts an Amount and returns another ChargeCommand.

For handling this query, we get the user's balance from Stripe. We pass that to
`balanceToCommand`, which constructs the next command we need to interpret.
We'll call `chargeCommand` again to handle the command that we just got.


## Interpreting 

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class UserBalanceStripe
  def run(command)
    user = command.user
    balance = user.balance?
    next_command = command.do_next.call(balance)
    return next_command
  end
end
</code></pre></div>

Note:

Back to Ruby!

For the user balance, We extract the user from the command, call Stripe to get
the user's balance, and pass the balance to the command's "do_next" method.
Finally, we return the command that is generated from this callback


## Locally Testing

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class UserBalanceLocal
  attr_reader :balances

  def initialize(balances)
    @balances = balances
  end

  def run(command)
    balance = balances[command.user.id]
    command.do_next(balance)
  end
end
</code></pre></div>

Note:

We can also create a mock interpreter, that lives locally.
Here we initialize it with a mapping from user IDs to their balances, and when we receive the command, we just check that hash map.  


## Locally Testing

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
chargeLocally 
  :: ChargeCommand a
  -> State (Map UserId Amount) a
chargeLocally (UserBalance user andThen) = do
  balance <- gets (Map.lookup (userId user))
  chargeLocally (andThen balance)
</code></pre></div>

Note:

The Haskell is basically the same thing, just with different syntax.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class ChargeStripe
  def run(command)
    price = command.subscription.price
    command.user.charge!(price)
    command.do_next.call(nil)
  end
end
</code></pre></div>

Note:

For charging against stripe, it's basically the same thing. We call the charge
method on the command's user, and then generate the next command in the
sequence. We'll pass `nil` to that callback, since there's no meaningful value here.


# Put it all together

Note:

Now, our functions are returning the next thing in the sequence.
But we haven't put them all together yet.
We need to be able to run these in sequence.
The Haskell case is easy -- since all the types are just defined as alternatives, we just write a single function.
The Ruby is a little harder.


## Reducing Command Trees

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class Interpeter
  attr_reader :handlers

  def initialize(handlers)
    @handlers = handlers
  end

  # ...
</code></pre></div>

Note:

This can be a bit tricky to think about. Like the ComposeCommand class we made
earlier, we've got our hash of handlers that we're initialized with. 


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
  # ...

  def interpret(command)
    <span class="fragment">loop do
      <span class="fragment">return command.result if Return === command</span>

      <span class="fragment">handler = handlers[command.class]
      command = handler.run(command)</span>
    end</span>
  end
end
</code></pre></div>

Note:

We have two cases: If the command is a Return, then we terminate execution and return whatever the result of the command is.
Otherwise, we get our handler for the given command, and run it with the command.
This gives us a next command to work with.
We then recurse on the function, calling `interpret` on the next command.


# Grafting Trees

Note:

There's one last piece of the puzzle.
We need a way to compose two command trees.
That is, given some command sequence, we need to have a way to say "And after you do all of this stuff, run this other sequence of commands."
That'll give us some nicer syntax, and a way of combining smaller commands into larger ones.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
pure = -> (x) { Return.new(x) }

def charge_or_email(user, subscription)
  UserBalance.new(user, -> (balance) do
    if balance >= subscription.price
      Charge.new(
        user, subscription.price, pure
      )
    else
      SendBalanceNotice.new(
        user, subscription, pure
      )
    end
  end)
end
</code></pre></div>

Note:

Here's our command structure for deciding whether to charge or email a user.
Now, we need some way to compose it, so we can iterate over all of the users
subscriptions and run this command tree.

Essentially, we want to say: "When you're done executing this plan, execute this next plan"


# Return = Terminate

Note:

We terminate execution when we use the Return command.
So we should be able to bind two trees together by replacing the Returns in the original tree with the sequences we're building out.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
def and\_then(cmd, after)
  <span class="fragment">case cmd
  when Return
    after.call(cmd.result)</span>
  <span class="fragment">when Charge
    Charge.new(
      cmd.user, cmd.price, -> x do
        and\_then(cmd.do\_next.call(x), after)    
      end
    )
  # ... etc</span>
</code></pre></div>

Note:

Or, in other words, we want to be able to say "do these commands, *and then* do these other commands."

For the Return case, we have it easy: we just provide the Return value of the command sequence to the after callback.
That replaces the Return value with a new sequence of commands.

For the Charge case, we create a new Charge object with the same user and price.
The callback is different, though -- we pass the value from the callback first into the old command's "do_next" callback.
This gives us a new command value.
Then we call 'and then' recursively on that new command tree.
Eventually, this should bottom out and hit a Return.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
  when UserBalance
    UserBalance.new(
      cmd.user, -> (balance) do
        and_then(
          cmd.do_next.call(balance), after
        )
      end
    )
  # ...
end
</code></pre></div>

Note:

The query uses a similar pattern. We create a new UserBalance command, and in
the callback, we pass the balance to the original command's callback, and
recurse into `and_then`.

In OOP, if you find yourself casing on classes, it's typically a good idea to put those as methods on the class.
So let's put that stuff on the class!


keep it classy

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class UserBalance < Value(:user, :do_next)
  <span class="fragment">def and_then(&callback)
    <span class="fragment">self.with(do_next: <span class="fragment">-> (balance) do
      <span class="fragment">self.do_next.call(balance)</span>
          <span class="fragment">.and_then(callback)</span>
    end</span>)</span>
  end</span>
end
</code></pre></div>

Note:

The Values library allows us to extend values by subclassing them.
Now, we can add a method directly to the UserBalance value object.
We can also make *copies* of these values by using a with method.
This creates a new copy of the object, but with the provided value changed.
Here we're going to update the do_next property on this object.

We call *self*'s do_next callback with the balance.
Then we call 'and_then' on the resulting value.


# The Scary 'M' Word

<h1 class="fragment"> Module</h1>

Note:

It turns out, none of this depends on any specific details of the class.
That means we can extract it out to the scary 'M' word: MODULE!


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
module Command
  def and_then(&callback)
    self.with(do_next: -> (value) do
      self.do_next.call(value)
          .and_then(callback)
    end)
  end
end
</code></pre></div>

Note:

Now, we can simply include this module in all of our commands, and we get the
and_then method for free.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class Charge < Value(:user, :amount, :do_next)
  include Command
end

class UserBalance < Value(:user, :do_next)
  include Command
end

class Return < Value(:result)
  def and_then(&block)
    block.call(self.result)
  end
end
</code></pre></div>

Note:

So here's how we define our command types. Return gets a special case --
instead of chaining the next command to the do-next parameter, we just pass
the result to the block directly.

I don't know about you, but I also kinda want some syntax sugar.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
pure = -> (x) { Return(x) }

def user_balance(user)
  UserBalance.new(user, pure)
end

def charge(user, amount)
  Charge.new(user, amount, pure)
end

def send_balance_notice(user, subscription)
  SendBalanceNotice.new(user, subscription, pure)
end
</code></pre></div>

Note:

The basic command terminates with a Return, so we'll terminate our singleton commands with this pure function.
The other helpers just make a new object of the right command class, and start it off with the Return constructor.
When we go to implement more of this stuff, we'll use the `and_then` method to build the chain.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
def charge_user(user)
  cmd = user_balance(user)

  <span class="fragment">user.subscriptions.each do |sub|
    <span class="fragment">cmd = cmd.and_then do |balance|
      <span class="fragment">if balance >= sub.price
        charge(user, sub.price)
      <span class="fragment">else
        send_balance_notice(user, sub)</span>
      end</span>
    end</span><span class="fragment">.and_then do
      user_balance(user)
    end</span>
  end</span>

  <span class="fragment">return cmd</span>
end
</code></pre></div>

Note:

Alright, so let's see what this code looks like now.
We'll start off by checking the user's balance.

Then, we'll iterate over the user's subscriptions.
For each subscription, we'll extend the command.
If the user's balance is OK, then we charge them.
Otherwise, we send a notice.
We extend that command with another balance check.

Finally, we return our command tree.

This structure perfectly encodes the logic we want, and it won't be evaluated
until we actually run it. So we've successfully created an execution plan.

The syntax is a little nicer now -- you're pretty close to something
idiomatically Ruby. I bet we can get nice syntax in Haskell, too.


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
data ChargeCmd a
  = Charge User Amount (ChargeCmd a)
  | UserBalance User (Amount -> ChargeCmd a)
  | SendBalanceNotice User Sub (ChargeCmd a)
  | Return a

andThen 
  :: ChargeCmd a 
  -> (a -> ChargeCmd b) 
  -> ChargeCmd b
andThen = ...
</code></pre></div>

Note:

Some of the nicer DSLs in Haskell use `do` syntax. If we weant `do`, then we
need to make this type an instance of Monad. 
It turns out, the `and_then` method we defined is almost exactly the code we need for our monad instance.


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
instance Monad ChargeCmd where
  (>>=) :: ChargeCmd a 
        -> (a -> ChargeCmd b) 
        -> ChargeCmd b
  command >>= makeCommand =
    <span class="fragment">case command of</span>
      <span class="fragment">Return a ->
        makeCommand a</span class="fragment">
      <span class="fragment">Charge user amount next ->
        Charge user amount (next >>= makeCommand)</span>
      <span class="fragment">UserBalance user next ->
        UserBalance user (\balance ->
          next balance >>= makeCommand)</span>
      <span class="fragment">SendBalanceNotice user sub next ->
        SendBalanceNotice user sub (next >>= makeCommand)</span>
</code></pre></div>

Note:

So here's our monad instance. The weird symbol function up there is pronounced
"bind" or "and then", and it takes a ChargeCmd as it's first argument. The
second argument is a function which accepts an "a" value, returning a ChargeCmd
b. The return is a ChargeCmd b.

Our base case is the Return constructor.
Since we have a value of type `a` handy, and a function from a to a ChargeCommand, we can just apply that function to our value.

For a Charge command, we'll want to create a new charge command with the same stuff, but we'll bind our makeCommand to the next parameter.

For the UserBalance command, we need to handle passing that Amount correctly.
Otherwise, it's really similar!

We recursively dig deeper and deeper into the command structure, until we
eventually find a 'Return' constructor. Then we apply the callback to the value
contained there.  Let's look at our program in Haskell now, to see how we can
take advantage of the new syntax.


## The Helpers

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
userBalance :: User -> ChargeCmd Amount
userBalance user = 
  UserBalance user (\balance -> Return balance)

charge :: User -> Amount -> ChargeCmd ()
charge user amount = 
  Charge user amount (Return ())

sendBalanceNotice 
  :: User -> Subscription -> ChargeCmd ()
sendBalanceNotice user subscription =
  SendBalanceNotice user subscription (Return ())
</code></pre></div>

Note:

These guys are going to help our syntax look nice, by presenting a way of lifting our command data structure into a more ordinary function call, easily compatible with the do notation.


# YEAAHHHH

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
chargeSubscriptions :: User -> ChargeCmd ()
chargeSubscriptions user =
  for_ (userSubscriptions user) $ \sub -> do
    balance <- userBalance user
    let price = subscriptionPrice sub
    if balance >= price
      then charge user price
      else sendBalanceNotice user sub
</code></pre></div>

Note:

Oh yeah!!! This is awesome. We're using entirely normal Haskell syntax to
express our business logic, without having to do anything too weird or fancy. 
Indeed, this looks almost exactly like the code we'd run in IO, but instead of
running directly in IO, we're just constructing a data structure.


# Intrigued?

## It's a free monad <!-- .element: class="fragment" -->

Note:

If you want to learn more about this final evolution of the command pattern,
the theory behind it is called the free monad. It's super interesting and very
powerful. 


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
data ChargeCmdF next
  = Charge User Amount next
  | UserBalance User (Amount -> next)

<span class="fragment">data Free f a 
  = Free (f (Free f a)) 
  | Return a</span>

<span class="fragment">instance (<span class="fragment">Functor f</span>) => Monad (Free f) where
  fa >>= k = case fa of
    Free f -> Free (fmap (>>= k) f)
    Return a -> k a</span>
    
<span class="fragment">instance Functor ChargeCmdF where
  fmap f cmd = case cmd of
    Charge user amount next ->
      Charge user amount (f next)
    UserBalance user next ->
      UserBalance user (f . next)</span>
</code></pre></div>

Note:

First, we factor out the business of "what to do next" from our command type.
This gives us this Free data type.
All we need to give Free a monad instance is a Functor instance: that's why we call it "The free monad for a functor".
We can define functor instances for command types like this -- in fact, this functor instance is boilerplate!
GHC can even derive it for us.

Unfortunately, most typed languages can't express it well. Haskell
and Scala both have great free monad libraries, but Java/F#/C#/etc. are unable
to express these things.
