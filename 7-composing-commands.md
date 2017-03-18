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

Ruby lets you use case..when syntax against the classes we're comparing against.
So when the command is a Subscribe (or a subclass of Subscribe), we'll use the
Stripe Subscriber. When it's Cancel, we'll call the stripe canceller. Otherwise
we throw an error.


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
structure that accepts a mapping of commands to their handlers.


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


```ruby
class ComposeInterpreters
  attr_reader :handlers

  def initialize(handlers)
    @handlers = handlers
  end

  def call(command)
    handlers[command.class].call(command)
  end
end
```

Note:

The implementation is even a lot simpler. And we can easily compose existing
composed commands using Ruby's hash merge methods:


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


# Another kind of composition?

Note:

This covers a way to extend interpreters to handle multiple distinct commands.
What if we also want to assign multiple handlers for a command? Suppose we want to additionally *log* every subscription and cancellation that happens.


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
  def initialize(interpreters)
    @interpreters = interpreters
  end

  def call(command)
    @interpreters[command.class].map do |interpreter|
      interpreter.call(command)
    end
  end
end
```

Note:

Now, when we're composing commands, we can still do hash merge, but we'll want
to control how merging occurs. By default, merging hashes in Ruby considers a
collision to be an update. Instead, we want it to be list append.


```
def compose_handlers(commands_1, commands_2)
  commands_1.merge(commands_2) do |_, c_1, c_2|
    c_1 + c_2
  end
end
```

Note:

Now, we can easily merge our logging command with our other commands.


```ruby
stripe_handler = {
  Cancel    => [StripeCanceller.new, Logger.new],
  Subscribe => [StripeSubscriber.new, Logger.new]
}
```
