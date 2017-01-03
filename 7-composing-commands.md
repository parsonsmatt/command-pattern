# Distinct Commands

```ruby
Subscribe = Struct.new(:user, :plan)
Cancel    = Struct.new(:subscription)

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


# Composing Commands


```ruby
class StripeCancel
  def run(command)
    Stripe::Subscription.cancel(
      command.subscription
    )
  end
end

class StripeSubscribe
  def run(command)
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
  def run(command)
    case command
    when Subscribe
      StripeSubscribe.new.run(command)
    when Cancel
      StripeCancel.new.run(command)
    end
  end
end
```

Note:

Ruby lets you use case..when syntax against the classes we're comparing against. So when the command is a Subscribe (or a subclass of Subscribe), we'll use the Stripe Subscriber. When it's Cancel, we'll run the stripe canceller.


```ruby
interpreter = CancelOrSubscribe.new

commands = [
  Cancel.new(:foo),
  Cancel.new(:bar),
  Subscribe.new(:lol, :baz)
]

commands.map do |command|
  interpreter.run command
end
```

Note:

Here's how we solve the problem of multiple classes in our command array.


```ruby
class CancelOrSubscribe
  attr_reader :canceller, :subscriber
  def initialize(canceller, subscriber)
    @canceller  = canceller
    @subscriber = subscriber
  end

  def run(command)
    case command
    when Subscribe
      subcriber.run(command)
    when Cancel
      canceller.run(command)
    end
  end
end
```

Note:

We can make this more generic by passing the specific canceller and subscriber
in. This is sort of like dependency injection or object composition.


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


```ruby
stripe_manager = ComposeCommands(
  {
    Cancel => StripeCanceller,
    Subscribe => StripeSubscriber
  }
)
```

Note:

Now, this is easy! We just pass a simple hash in, and our composition is done.


```ruby
class ComposeCommands
  attr_reader :handlers

  def initialize(handlers)
    @handlers = handlers
  end

  def run(command)
    handlers[command.class].run(command)
  end
end
```

Note:

The implementation is even a lot simpler. And we can easily compose existing composed commands using Ruby's hash merge methods:


```ruby
stripe_handlers = { 
  Cancel => StripeCanceller.new,
  Subscribe => StripeSubscriber.new 
}
user_handlers = {
  Delete => StripeUserDelete.new,
  Create => StripeUserCreate.new
}

stripe_manager = ComposeCommands(
  stripe_handlers.merge(user_handlers)
)
```

Note:

So, this gives us the tools to easily compose command handlers from very simple
classes.
