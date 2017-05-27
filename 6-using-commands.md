# Interpreting Commands

Note:

Now that we've got all these commands, how do we actually use them?
There are a number of ways we can do this, in increasing complexity/power.
As with most things in software development, it's best to ask for as little power as you need.
Shooting yourself in the foot hurts a lot less if you only have a water gun.


# Command Interpreter

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class FooResultInterpreter
  def call(command)
    FooResult.insert(
      command.x, command.y, command.z
    )
  end
end
</code></pre></div>

Note:

For each command, you implement at least one interpreter.
Here's a basic interpreter for the FooResult class.


# Command Interpreter

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
fooResultInterpreter :: InsertFooResult -> DB ()
fooResultInterpreter (InsertFooResult x y z) = 
    insert (FooResult x y z) 
</code></pre></div>

Note:

Command interpreters in Haskell are just functions which convert a *command value* into an action of some sort.


# Single Command

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
value, action = Foo.new.my_func(1,2,3)

InsertFooResultInterpreter.new.call(action)
# or,
TestInsertFooResultInterpreter.new.call(action)
# or,
RedisFooResult.new.call(action)
</code></pre></div>

Note:

So, if you've got a single command, this is the easiest. You take that object
and you pass it to the interpreter of your choice. You can easily define many
different varieties of interpreters for a given command.


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
let (value, action) = Foo.myFunc 1 2 3

insertFooResult     
  :: InsertFooResult -> DB ()

testInsertFooResult 
  :: InsertFooResult -> State MockDB ()

redisFooResult      
  :: InsertFooResult -> Redis ()
</code></pre></div>


# Back to Billing Logic

Note:

OK, that's great, but let's get back to the important stuff -- getting money
from people!


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
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
</code></pre></div>

Note:

These classes share the same interface and can be used interchangeably.
So we can easily pass a single command to them and have it be executed.
Likewise, since they have the same duck type, we can pass any old instance of a
subscriber interpreter to a class.


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
type SubscribeInterpreter m = 
  SubscribeCommand -> m ()

stripeSubscriber   
  :: SubcribeInterpreter IO
internalSubscriber 
  :: SubscribeInterpreter IO
testSubscriber     
  :: SubscribeInterpreter (State User)
</code></pre></div>

Note:

The Haskell shape, or interface, is going to be parameterized over the kinds of effects it can do.
A Stripe subscriber or internal billing system is going to operate in IO, while a test interpreter might work in a local in memory state containing a User.


# Running Multiple Commands

Note:

That covers the single command case, like our toy example. What about multiple
commands, like our more involved refactoring example? Since we're dealing with
simple values, we can use simple functions to chain commands. We'll start with
a simple case: multiple of the same command.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
<br/>
commands = [
  Subscribe.new(x, y), 
  Subscribe.new(z, a),
  Subscribe.new(b, c)
]

# answer?
# wat do
# halp
</code></pre></div>

Note:

Ok, so we've got an array full of Subscribe commands. And we want to actually execute each one.
Does anyone have a suggestion on how to do this?


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
interpeter = StripeSubscriber.new

commands = [
  Subscribe.new(x, y), 
  Subscribe.new(z, a),
  Subscribe.new(b, c)
]

commands.map do |command| 
  interpreter.call command
end
</code></pre></div>

Note:

We can simply map over the commands, and for each command, call it through the interpreter. 


# Functor

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
executeCommands
  :: [SubscribeCommand]
  -> <span class="fragment">(SubscribeCommand -> IO a)</span>
  -> <span class="fragment">[IO a]</span> <span class="fragment">-- Not the \`IO [a]\` that we want :(</span>
executeCommands commands interpreter =
  map interpreter commands
</code></pre></div>

Note:

Well, unfortunately `map` just doesn't quite handle it like we'd want to.
In Ruby, this would iterate over each item in the array, execute the callback, and return the array of all results.
In Haskell, what we get is a list of IO operations -- nothing actually happens here, we just prepare the instructions to be executed later.
We're looking for something a little more powerful.


# Traversable

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
executeCommands
  :: [SubscribeCommand]
  -> (SubscribeCommand -> IO a)
  -> IO [a]
executeCommands commands interpreter =
  traverse interpreter commands
</code></pre></div>

Note:

In Haskell, if you want to iterate over a collection along with some effect, then you want to use Traversable.
A Haskell traversal is like an effectful map, or what you're used to seeing in non-pure languages.


# Simple <span class="fragment">Data</span>

# Is Easy

Note:

Commands are just simple data. Since they're simple data, we can easily stuff them
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


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
commands = [
  InsertFooResult.new(1, 2, 3),        <span class="fragment"># one insert...</span>
  DoSomeOtherThing.new("hello world"),
  InsertFooResult.new(4, 5, 6),        <span class="fragment"># two inserts...</span>
  InsertFooResult.new(7, 8, 9),        <span class="fragment"># three inserts...</span>
  LaunchTheMissiles.new(Time.now)     
]

commands.map { |cmd| interpreter.call(cmd) }
</code></pre></div>

Note:

Now, we've got this set of commands we want to handle.
We're inserting a bunch of rows into the database and doing some other stuff.

Here's our naive logic. It *works*. But it's inefficient! We issue three SQL
queries here, when we could save a tremendous amount of time by doing a bulk
insert.

Let's optimize this.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
def optimize(commands)
  inserts, rest = commands.partition do |command|
    command === InsertFooResult
  end

  rest.push(BulkInsert.new(inserts))
end
</code></pre></div>

Note:

Ok, so here we're going to partition the commands into two lists; the first is
one where the command's class is an InsertFooResult. The second list is all of
the other commands. We return the list of non-insert commands with a BulkInsert
command appended to the end.
