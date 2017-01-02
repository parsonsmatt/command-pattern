# Using Commands

Note:

Now that we've got all these commands, how do we actually use them? There are
a number of ways we can do this, in increasing complexity/power. As with most
things in software development, it's best to use as little power as you need.
Shooting yourself in the foot hurts a lot less if you only have a water gun.


# Single Command

```ruby
class Foo
  def lawl(x, y, z)
    x + y + z, InsertFooResult.new(x, y, z)
  end
end

value, action = Foo.lawl(1,2,3)
action.run
# or,
InsertFooResultInterpreter.new.run(action)
```

Note:

So, if you've got a single command, this is the easiest. You take that object
and you pass it to the interpreter of your choice. Either one defined on the
Command class itself, or on a separate interpreter class. They're really
equivalent -- it's easier to use separate interpreters if you anticipate having
multiple ways to interpret a given command, though you can use
inheritance/subclassing to mimic that with the class.


# Built In

```ruby
class Subscribe
  attr_reader :user, :plan

  def initialize(user, plan)
    @user, @plan = user, plan
  end

  def run
    Stripe::Subscription.create(user, plan)
  end
end
```

Note:

The built in method allows us to keep all of the info in one place. This is
good. But it limits how we're executing the effects.


```ruby
Subscribe = Struct.new(:user, :plan) do
  def run
    Stripe::Subscription.create(user, plan)
  end
end
```

Note:

You can give structs methods, which is a nice shorthand. It makes it a little
more concise and easy to read. However, you can't really subclass these, and
you don't get an error if you don't pass all the parameters. So you may end up
growing out of the struct style.


```ruby
class OtherSubscribe < Subscribe
  def run
    InternalBilling.create_subscription(
      user, plan
    )
  end
end
```

Note:

With the built in class, you can vary behavior with inheritance. In this
example, we're subclassing the Subscribe class we created above and overriding
the `run` method. However, now our calling code has to issue this
`OtherSubscribe` method, which is intermixing our concerns.


# Coupling :(


# Separate

```ruby
class StripeSubscriber
  def run(command)
    Stripe::Subscription.create(
      command.user, command.plan
    ) 
  end
end
```

Note:

With a separate runner class, we can easily define alternative interpretations.


```ruby
class InternalSubscriber
  def run(command)
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

commands.each do |command| 
  interpreter.run command
end
```

Note:

We can simply iterate over the commands, and for each command, run it through
the interpreter. There's a downside to this, though. If there's a significant
return value of the command, then it's lost. We can retain those return values
by using map instead.


```ruby
interpreter = StripeSubscriber.new
commands = [
  Subscribe.new(x, y), 
  Subscribe.new(z, a),
  Subscribe.new(b, c)
]

commands.map do |command| 
  interpreter.run command
end
```

Note:

So that's the next easiest case. We take our method which works on a single
command, and then we map it over a collection of commands. Let's kick the
difficulty up a notch. What if we have two different command types?
