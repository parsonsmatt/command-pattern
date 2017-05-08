# LEVEL UP

Note:

Alright, let's level up our Ruby command interpreter. Haskell has a leg up,
since it can have sum types. But we want to evaluate different commands in
Ruby, too! How do we do that?


# Distinct Commands

```ruby
Subscribe = Value.new(:user, :plan)
Cancel    = Value.new(:subscription)

commands = [
  Cancel.new(:foo),
  Cancel.new(:bar),
  Subscribe.new(:lol, :baz)
]

# wat do?
```

Note:

We can't *just* map over this with our StripeSubscriber, because it doesn't
know how to handle Cancel commands. Fortunately, it's real easy to compose two
command interpreters.


# Composing Interpreters


```ruby
class StripeCancel
  def call(command)
    Stripe::Subscription.cancel(
      command.subscription
    )
  end
end

class StripeSubscribe
  def call(command)
    Stripe::Subscription.create(
      command.user, command.plan
    )
  end
end
```

Note:

Here are our cancel and subscribe command interpreters. Super basic, no logic,
easy to read, test, understand, etc. Now let's compose them:


# For want of a sum...

```haskell
data SubscribeCommand
  = Subscribe User Plan
  | Cancel Subscription

stripeInterpreter :: SubscribeCommand -> IO ()
stripeInterpreter (Subscribe user plan) = ...
stripeInterpreter (Cancel subscription) = ...
```

Note:

Haskell's sum types make it easy to concretely encode this.  We just provide a
sum type with all the constructors of our command, and then we write a function
accepting that type as an argument. If we cover all the cases, then we're done.


```ruby
class CancelOrSubscribe
  def call(command)
    case command
    when Subscribe
      StripeSubscribe.new.call(command)
    when Cancel
      StripeCancel.new.call(command)
    else
      raise UnkownCommandError.new(command)
    end
  end
end
```

Note:

Ruby lets you use case..when syntax against the classes we're comparing
against.  So when the command is a Subscribe (or a subclass of Subscribe),
we'll use the Stripe Subscriber. When it's Cancel, we'll call the stripe
canceller. Otherwise we throw an error. This looks a lot like the Haskell code
-- we know all of the possible commands while writing the code, we case on the
constructor for the command, and we delegate to that interpreter's
implementation.


```ruby
interpreter = CancelOrSubscribe.new

commands = [
  Cancel.new(:foo),
  Cancel.new(:bar),
  Subscribe.new(:lol, :baz)
]

commands.map do |command|
  interpreter.call command
end
```

Note:

Here's how we solve the problem of multiple classes in our command array.
Now that we have a *composed* interpreter, it's easy to do this.


```ruby
class CancelOrSubscribe
  def initialize(canceller, subscriber)
    @canceller  = canceller
    @subscriber = subscriber
  end

  def call(command)
    case command
    when Subscribe
      @subcriber.call(command)
    when Cancel
      @canceller.call(command)
    end
  end
end
```

Note:

We can make this more generic by passing the specific canceller and subscriber
in.


```haskell
-- | Dependency Injection! Behold the 
-- enterprisey glory of this function.
genericCancelOrSubscribe 
  :: (SubscribeCommand -> IO ()) 
  -> SubscribeCommand -> IO ()
genericCancelOrSubscribe action command = 
  action command
-- or, simply,
genericCancelOrSubscribe = 
  id
```

Note:

This sort of dependency injection doesn't make all that much sense in Haskell, though.


# Interpeter Duck

```ruby
stripe_manager = CancelOrSubscribe.new(
  StripeCanceller.new, StripeSubscriber.new
)

internal_manager = CancelOrSubscribe.new(
  InternalCanceller.new, InternalSubscriber.new
)
```

Note:

Now, it's pretty easy to construct interpreters for sets of commands.
However, I'm still not entirely happy with this. I want to describe a generic
structure that accepts a mapping of command types to their handlers.


### insert bird law joke here


```ruby
stripe_manager = ComposeInterpreters.new(
  {
    Cancel => StripeCanceller.new,
    Subscribe => StripeSubscriber.new
  }
)
```

Note:

Now, this is easy! We just pass a simple hash in, and our composition is done.
This is actually a little difficult to accomplish in Haskell -- if we fix the
type of commands in advance, then we can make a simple sum type, and just
interpret that normally.  However, if we want to be able to flexibly combine
commands and interpreters (of differing types), then we're going to have a hard
time. We'd have to dig into some interesting type class and type programming
stuff.

Let's see how this can be implemented, in Ruby at least.


```ruby
class ComposeInterpreters
  def initialize(handlers)
    @handlers = handlers
  end

  def call(command)
    handler = @handlers[command.class]
    if handler
      handler.call(command)
    else 
      raise UnkownCommandError.new(command)
    end
  end
end
```

Note:

The implementation is even shorter. We dig into the dictionary we're passed
with the class of the command that we're given, and we call the handler with
the command if it's present. Otherwise we throw a runtime error. 


# Composing 

# Composing

Note:

We can also easily compose these new composed handlers.


```ruby
stripe_handlers = { 
  Cancel    => StripeCanceller.new,
  Subscribe => StripeSubscriber.new 
}
user_handlers = {
  Delete => StripeUserDelete.new,
  Create => StripeUserCreate.new
}

stripe_manager = ComposeInterpreters.new(
  stripe_handlers.merge(user_handlers)
)
```

Note:

Here we have two hashes of handlers. One handles subscriptions, with a canceller
and a subscriber. The other handles users, with a delete and create command. To
compose them, we can just use Ruby's hash merge method. 

We can consider this a *widening* of the original command handler.

Hash merge forms a monoid, so the widening of a command interpreter *also*
forms a monoid.  These things are everywhere, even in your Ruby or PHP code.


# Another kind of composition?

Note:

This covers a way to extend interpreters to handle multiple distinct commands.
What if we also want to assign multiple handlers for a command? Suppose we want to additionally *log* every subscription and cancellation that happens.


# A New Interpreter

```ruby
class CommandLogger
  def call(command)
    puts command
  end
end
```

Note:

How can we combine our handlers? We extend our handlers to require arrays of handlers, not just a single one.


```ruby
class ComposeInterpreters
  def initialize(interpreters); @interpreters ...

  def call(command)
    handler = @interpreters[command.class]

    unless handler 
      raise UnkownCommandError.new(command) 
    end
     
    handler.map do |interpreter|
      interpreter.call(command)
    end
  end
end
```

Note:

This implementation of the ComposeInterpeters class expects the various
handlers to be an *array* of interpreters. The return value that we get from
this composed interpreter is then an array of the results of calling the
various handlers.


## Monoid of Monoids

```ruby
def compose_handlers(commands_1, commands_2)
  commands_1.merge(commands_2) do |_, c_1, c_2|
    c_1 + c_2
  end
end
```

Note:

Now, when we're composing commands, we can still do hash merge, but we'll want
to control how merging occurs. By default, merging hashes in Ruby considers a
collision to be an update. Instead, we want it to be list append.

With this change, we can easily merge our logging command with our other commands.


#### so convenient

```ruby
def with_logging(handlers) 
  handlers.reduce(Hash.new) do |acc, x|
    klass, handler = x
    acc.merge(
      { klass => handler + [Logger.new] }
    )
  end
end

stripe_handler = with_logging({
  Cancel    => [StripeCanceller.new],
  Subscribe => [StripeSubscriber.new]
})
```

### wow

Note:

Now we can easily enhance our command handlers with new functionality. Since
these are just dumb data structures, it's prety easy!


# Haskellize?

Note:

So I've shown a lot of Ruby code on this last part. The truth is, this kind of
dynamic extensibility -- where you can easily add new command interpeting
functionality -- is best done by adding new constructors to a sum type. 


# but but 

### (the boiler plate?!) <!-- .element: class="fragment" -->

the expression PROBLEM <!-- .element: class="fragment" -->

Note:

But, you know, isn't there boilerplate for this?
Isn't this the Expression Problem that so plagues programmers?
Shouldn't we be going into Data Types A La Carte programming?

Nah. The extra complexity is rarely worth it. If you're an application
programmer, you get *far* more utility out of using the least powerful solution
to your problem.


```haskell
data SubscribeCommand
  = Subscribe User Plan
  | Cancel Subscription
  | CreateUser Name (Maybe Card) -- new!
  | DeleteUser User              -- new!

stripeInterpreter
  :: SubscribeCommand -> IO ()
stripeInterpreter command = case command of
  Subscribe user plan -> ...
  Cancel sub -> ...
  -- we get compile warnings if we forget these!
  CreateUser name maybeCard -> ...
  DeleteUser user -> ...
```

Note:

When we add these new commands to our sum type, we get compile warnings if we
fail to handle them in any use case. And *that's* awesome.


# Composing Handlers?

```haskell
type SubscribeHandler 
  = SubscribeCommand -> IO ()

runManyHandlers 
  :: [SubscribeHandler] 
  -> SubscribeCommand 
  -> IO ()
runManyHandlers handlers command = 
  sequence handlers command
```

Note:

We can still compose multiple handlers. Given a list of SubscribeHandlers and a
single SubscribeCommand, we can use the sequence function to run each handler
on the command.


```haskell
-- Explicit, specialized:
runManyHandlers
  :: [SubscribeCommand -> IO ()]
  -> [SubscribeCommand]
  -> IO ()
runManyHandlers handlers commands =
  traverse_ (sequence handlers) commands

-- Generalized:
runManyHandlers
  :: (Foldable t, Traversable f, Applicative f)
  => f (a -> b)
  -> t a
  -> f ()
runManyHandlers = traverse_ . sequence
```

Note:

Running multiple handlers over multiple commands is, in truth, a pretty
simple operation. It's the composition of two functions!
