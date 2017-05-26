# Methods and Functions

## the unit of

### uh, doing *stuff*

Note:

So, when we talk about "doing stuff" in computer programming, we're typically talking about methods or functions.


# Input and Output

# Values and Effects
<!-- .element: class="fragment" -->

Note:

In order to reason about our programs, we have to be able to talk about them.
How can we talk about our functions?
Generally, we can talk about a function in terms of the inputs and outputs that it has.
We can also talk about a function in terms of the values and effects that it deals with.


![](1000px-Ruby_logo.svg.png) <!-- .element: id="ruby-logo", style: "size: 100" --> ![](haskell_logo.svg) <!-- .element: id="haskell-logo" -->

<3

Note:

The code in this talk will be a combination of Ruby and Haskell.
Ruby is a dynamically typed object oriented programming language.
Haskell is a statically typed pure functional programming language.
We'll get to see how the same technique works out pretty great in both languages!


# Values

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class Foo
  def my_func(x, y)            # value!
    z = User.all.length
    FooResult.insert(x, y, z)
    x + y + z                  # value!
  end
end
</code></pre></div>

* Input arguments
* Return value

Note:

The *values* in this code snippet are the simple, easy bits. We pass in the
numbers x and y, which are both values. We return x + y + z, which is also a simple
value.


# Effects

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class Foo
  def my_func(x, y)
    z = User.all.length       # effect!
    FooResult.insert(x, y, z) # effect!
    x + y + z
  end
end
</code></pre></div>

* Input effects
* Output effects

Note:

The effects in this function are reading all of the users out of the database,
and inserting a new result into the database.


# Haskell

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
module Foo where

myFunc :: Int -> Int -> DB Int
myFunc x y = do
    z <- fmap length selectUsers
    insert (FooResult x y z)
    pure (x + y + z)
</code></pre></div>

Note:

Haskell's purity makes it really easy to figure out what's an effect and what's
a value. In Ruby (or any other language, really) you have to either *know* or
read the entire call graph of the code you're talking about. Haskell tracks it
in the type, which makes these refactors really easy.


# Values are *explicit*

# Effects are *implicit*

Note:

Values are, at a first approximation, the things we pass directly into
functions or methods, and the things that are returned directly from functions.
Input effects are the things that provide information to the function that we
don't explicitly pass in. Output effects are the things that happen as a result
of calling the method, that aren't explicitly part of the return value.

So values are explicit. Testing values is easy, and testing is an OK
approximation of good software. Effects are implicit. And testing effects is difficult.


# Testing Values:

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
def add(x, y)
  x + y
end

# In some Spec.rb file, 

describe "add" do
  it "should add" do
    expect(add(2, 3)).to eq 5
  end
end
</code></pre></div>

Note:

So, here's the super simple add function. Testing it is super easy. We just
pass in some input values, and make assertions about the output value. This
test is pretty silly, but it's easy to come up with more advanced test cases.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
describe "Add" do
  it "is commutative" do
    100.times do
      x, y = 2.times { Random::rand }
      expect(x + y).to eq(y + x)
    end
  end

  it "is associative" do
    100.times do
      x, y, z = 3.times { Random::rand }
      expect((x + y) + z).to eq(x + (y + z))
    end
  end
end 
</code></pre></div>

Note:

Here, we're testing that add is associative and commutative. We're taking
a random sample of values, and ensuring that we can reorder our operations and
group them however we want. These tests are *easy* to write. They're fun,
almost. And they're kinda pretty! They look extremely close to the mathematical
definitions of associativity and commutativity.

This sounds trivial, but many difficult concepts in distributed systems involve
guaranteeing properties like commutativity. Making these properties easy to test
is important.


# In Haskell, too!

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
add x y = x + y

spec = 
  describe "add" $ do
    it "should add numbers" $ do
      add 2 3 \`shouldBe\` 5    
    prop "is commutative" $ \x y -> do
      add x y \`shouldBe\` add y x
    prop "is associative" $ \x y z -> do
      add x (add y z) \`shouldBe\` add (add x y) z
</code></pre></div>

Note:

The tests we have for the Ruby and Haskell are just about the same! It's a
little easier to write the tests in Haskell, but we've got basically the same
thing going on.


# Testing Effects:

### dun
<!-- .element: class="fragment" -->

### dun
<!-- .element: class="fragment" -->
### dun
<!-- .element: class="fragment" -->

Note:

So let's talk about testing effects. (dun dun dun)

Testing effects is a lot harder. And sometimes, it can get really messy.
So let's go through some of the ways that we might test effects. We'll start
out kinda messy and wrong, and we'll evaluate some of the solutions that are
common today.


# Testing Effects:

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class Foo
  def my_func(x, y)
    z = User.all.length       # effect!
    FooResult.insert(x, y, z) # effect!
    x + y + z
  end
end
</code></pre></div>

Note:

Testing effects is a lot harder. Suddenly we have to worry about what User is,
and what happens when we do FooResult. I'm going to evolve testing this example.


# Wrong

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
describe "Foo#my_func" do
  it "adds inputs" do
    expect(Foo.new.my_func(1,2)).to eq(3)
  end
end
</code></pre></div>

Note:

So this attempt is flat out wrong. However, on an uninitialized database with 0
users, it'll return the right answer. This is a *fragile* test, even though it
may pass sometimes. We also don't capture the effect of creating a FooResult in
the database.


# Wrong Haskell

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
describe "Foo.myFunc" $ do
  it "adds inputs" $ do
    Foo.myFunc 1 2 
      \`shouldReturn\` 
        3
</code></pre></div>

Note:

Haskell isn't going to protect us here. While we know that we have effects
going on in Foo.myFunc, that's all we know, and as long as we acknowledge that,
then GHC is satisfied. Since the correctness of this depends on something that
we are not tracking in the type, the type system can' help us!


# Slow

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
describe "Foo#my_func" do
  it "adds inputs" do
    User.insert(name: "Matt", age: 28)
    expect(User.all.length).to eq(1)
    expect(Foo.new.my_func(1,2)).to eq(4)
    x = FooResult.find_by(x: 1, y: 2, z: 1)
    expect(x).to_not be_nil
  end
end
</code></pre></div>

Note:

So this isn't wrong anymore. However, the test relies on the database state, and
has to do five SQL queries in order to verify the code. These tests are
monstrously slow and will kill your TDD cycle, in addition to being fragile and
annoying to write. Now you need to *also* write a bunch of database cleanup and
state management code for your tests. Gross.


# Stubs!

Note:

The next "level up" that often happens is to take advantage of stubs or mocks.
Let's look at that real quick:


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
describe "Foo" do
  it "adds some numbers" do
    x, y, z = 3, 4, 3
    <span class="fragment">expect(User)
      .to receive(:all)
      .and_return([1,2,3])</span>
    <span class="fragment">allow(FooResult)
      .to receive(:insert).with(x, y, z)</span>
    <span class="fragment">expect(Foo.new.my_func(x, y))
      .to eq(x + y + z)</span>
  end
end
</code></pre></div>

Note:

This test is a lot nicer. We start with some initial values we'll use. 
We need to stub out the User.all method to ensure it returns a value that suits our expectation.
We also need to stub out the FooResult class and verify that it receives the arguments we expect.
Finally, we can do some assertions on the actual values involved.

This kinda sucks! You can imagine extending this to more complex things, but it
gets even uglier, pretty quickly. Furthermore, stubs and mocks are pretty
controversial in the OOP community. They're not a clear best practice.


# Stubs in Haskell

Note:

So stubbing global terms like this in Haskell? It's not possible. Sorry, or
not, I guess, depending on whether you find the previous code disgusting or
pleasantly concise.

This is also not possible, or extremely difficult/annoying, in other langauges.
So, rather than sticking with something that only works with Ruby, lets find
something that we can use in most languages we might want to use.


# Dependency Injection?

Note:

Dependency injection is usually heralded as the solution or improvement to just
stubbing out random global names. You define an interface (or duck type) for
what your objects need and pass them in your object initializer


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class Foo
  <span class="fragment">def initialize(user, foo_result)
    @user = user
    @foo_result = foo_result
  end</span>

  def my_func(x, y)
    z = @user.all.length        # effect!
    result = x + y + z
    @foo_result.insert(x, y, z) # effect!
    result
  end
end
</code></pre></div>

Note:

So here, instead of referring to the global term User and FooResult, we refer
to instance variables.

We provide these values to our class on initialization.

Dependency injection can be used to make testing like this a little easier,
especially in languages that aren't as flexible as Ruby (like Haskell). Instead
of overriding a global name, you make the class depend on a parameter that's
local. Instead of referring to the global User class, we're referring to the
instance variable user which is ostensibly the same thing. This is Good, as
we've reduced the coupling in our code, but we've introduced some significant
extra complexity.  And the testing story isn't great, either:


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
describe "Foo" do
  it "adds stuff" do
    x, y, users = 2, 3, [1,2,3]
    <span class="fragment">user = UserTest.new(users)
    foo_result = FooResultTest.new

    foo = Foo.new(UserTest.new, FooResultTest.new)</span>

    <span class="fragment">expect(foo_result.values).to include([x, y, 3])
    expect(foo.my_func(x, y)).to eq(x + y + 3)</span>
  end
end
</code></pre></div>

Note:

To test a dependency injection thing, first we initialize some values.

Then we create our test versions of the dependencies, and initialize the foo class with these dudes.

Finally, we can make some assertions about the return value.

This is clearly worse than before. If we're going by the metric that easier to
test code is better code, then this code *really* sucks. We're dependent on
observing mutability and other complexities. So dependency injection is clearly
not the obvious solution to this problem, though it does make our code more extensible.


# How to even do that in Haskell

(it's just a function) <!-- .element: class="fragment" -->

Note:

So how do we even do that in Haskell? Well, everything is just a function, so you just pass functions.


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
<span class="fragment">myFuncAbstract 
    :: IO [a]               
    -- ^ select users
    -> (FooResult -> IO ()) 
    -- ^ insert FooResult
    -> Int -> Int -> IO Int 
    -- ^ The rest of the function</span>
myFuncAbstract <span class="fragment">selectUsers insert</span> x y = do
    z <- fmap length selectUsers
    insert (FooResult x y z)
    pure (x + y + z)

myFunc = myFuncAbstract DB.selectUsers DB.insert
</code></pre></div>

Note:

The type signature changes -- now, instead of doing the work directly, we defer to functions that we accept as parameters.

Here we make selectUsers and insert into functions that we pass in, and for the
real version, we provide the database functions. The tested version can provide 
different functions based on what you want to test.

This is about as awkward as the OOP Version! Alas.
There has to be something better.
If we could only make testing our effects as easy as testing our values... then we'd be set!
