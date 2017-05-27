# LEVEL UP

Note:

Alright, let's level up our Ruby command interpreter. Haskell has a leg up,
since it can have sum types. But we want to evaluate different commands in
Ruby, too! How do we do that?


# Distinct Commands

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
Subscribe = Value.new(:user, :plan)
Cancel    = Value.new(:subscription)

commands = [
  Cancel.new(:foo),
  Cancel.new(:bar),
  Subscribe.new(:lol, :baz)
]

# wat do?
</code></pre></div>

Note:

We can't *just* map over this with our StripeSubscriber, because it doesn't
know how to handle Cancel commands. Fortunately, it's real easy to compose two
command interpreters.


# Basic $\to$ Advanced

Note:

I think it's best to start simple, and only advance in power as you need it.
Complexity makes things hard, and simplicity is sometimes the right answer, even when it's verbose or repetitive.


# Composing Interpreters


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
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
</code></pre></div>

Note:

Here are our cancel and subscribe command interpreters. Super basic, no logic,
easy to read, test, understand, etc.


# For want of a sum...

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
data SubscribeCommand
  = Subscribe User Plan
  | Cancel Subscription

stripeInterpreter :: SubscribeCommand -> IO ()
stripeInterpreter command = 
  case command of
    Subscribe user plan ->
      Stripe.subscribe user plan
    Cancel subscription ->
      Stripe.cancel subscription
</code></pre></div>

Note:

Haskell's sum types make it easy to concretely encode this.
We just provide a sum type with all the constructors of our command, and then we write a function accepting that type as an argument.
If we cover all the cases, then we're done.

I bet we can get something similar with Ruby.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
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
</code></pre></div>

Note:

Ruby lets you use case..when syntax against the classes we're comparing against.
So when the command is a Subscribe (or a subclass of Subscribe), we'll use the Stripe Subscriber.
When it's Cancel, we'll call the stripe canceller.
Otherwise we throw an error.
This looks a lot like the Haskell code -- we know all of the possible commands while writing the code, we case on the constructor for the command, and we delegate to that interpreter's implementation.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
interpreter = CancelOrSubscribe.new

commands = [
  Cancel.new(:foo),
  Cancel.new(:bar),
  Subscribe.new(:lol, :baz)
]

commands.map do |command|
  interpreter.call command
end
</code></pre></div>

Note:

Here's how we solve the problem of multiple classes in our command array.
Now that we have a *composed* interpreter, it's easy to do this.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
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
</code></pre></div>

Note:

We can make this more generic by passing the specific canceller and subscriber
in.


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
-- | Dependency Injection! Behold the 
-- enterprisey glory of this function.
genericCancelOrSubscribe 
  :: (SubscribeCommand -> IO ()) 
  -> SubscribeCommand -> IO ()
genericCancelOrSubscribe action command = 
  <span class="fragment">action command</span>
<span class="fragment">-- or, simply,
genericCancelOrSubscribe = 
  id</span>
</code></pre></div>

Note:

This sort of dependency injection doesn't make all that much sense in Haskell, though.
Since almost everything is a simple function or a simple type, we can just write directly against it.


# Interpeter Duck

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
stripe_manager = CancelOrSubscribe.new(
  StripeCanceller.new, StripeSubscriber.new
)

internal_manager = CancelOrSubscribe.new(
  InternalCanceller.new, InternalSubscriber.new
)
</code></pre></div>

Note:

Now, it's pretty easy to construct interpreters for sets of commands.
However, I'm still not entirely happy with this.
I want to describe a generic structure that accepts a mapping of command types to their handlers.
That way, we won't have to rewrite the same machinery for every composition of commands.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
stripe_manager = ComposeInterpreters.new(
  {
    Cancel    => StripeCanceller.new,
    Subscribe => StripeSubscriber.new
  }
)
</code></pre></div>

Note:

This is the API I want to have.
This structure represents a mapping, or dictionary, where the keys in the dictionary are the command classes, and the values they point to are the handlers for that class.

Let's see how this can be implemented, in Ruby at least.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class ComposeInterpreters
  <span class="fragment">attr_reader :handlers 

  def initialize(handlers)
    @handlers = handlers
  end</span>

  <span class="fragment">def call(command)
    <span class="fragment">handler = handlers[command.class]
    if handler
      handler.call(command)</span>
    <span class="fragment">else 
      raise UnkownCommandError.new(command)
    end</span>
  end</span>
end
</code></pre></div>

Note:

First, we'll need to accept our dictionary from commands to interpreters.

When we implement call, the first thing we'll need to do is dig into the dictionary of handlers we had  with the class of the command that we're given.
If that command handler is present, then we call the handler with that command.
Otherwise, we throw a runtime error.


# Composing 

# Composing

Note:

We can also easily compose these new composed handlers.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class ComposeInterpreters
  def merge(other)
    @handlers = handlers.merge(other.handlers)    
  end
end

<span class="fragment">subscription_handlers = ComposeInterpreters.new({ 
  Cancel    => StripeCanceller.new,
  Subscribe => StripeSubscriber.new 
})

user_handlers = ComposeInterpreters.new({
  Delete    => StripeUserDelete.new,
  Create    => StripeUserCreate.new
})</span>

<span class="fragment">stripe_manager = 
  user_handlers.merge(subscription_handlers)</span>
</code></pre></div>

Note:

We'll add a merge method to composed handler class.
This lets us merge two ComposedInterpreters.
It just delegates to the internal Ruby hash merge method.

Here we have two hashes of handlers.
One handles subscriptions, with a canceller and a subscriber.
The other handles users, with a delete and create command.

Finally, we compose these two objects.
Hash merge forms a monoid, so the widening of a command interpreter *also* forms a monoid.
These things are everywhere, even in your Ruby or PHP code.


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


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
data SubscribeCommand
  = Subscribe User Plan
  | Cancel Subscription
  <span class="fragment">| CreateUser Name (Maybe Card) -- new!
  | DeleteUser User              -- new!</span>

stripeInterpreter
  :: SubscribeCommand -> IO ()
stripeInterpreter command = case command of
  Subscribe user plan -> ...
  Cancel sub -> ...
  <span class="fragment">-- COMPILE WARNING: Incomplete pattern match!</span>
  <span class="fragment">-- we get compile warnings if we forget these!
  CreateUser name maybeCard -> ...
  DeleteUser user -> ...</span>
</code></pre></div>

Note:

So here's our original code.
Aaaand, now we've exended it with two new command types for adding users!
When we add these new commands to our sum type, we get compile warnings if we
fail to handle them in any use case. And *that's* awesome.

This doesn't end up being *that* much boiler plate, in practice.
