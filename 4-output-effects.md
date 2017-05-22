# Output Effects

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class Foo
  def my_func(x, y, z)
    FooResult.insert(x, y, z)  # output effect!
    x + y + z
  end
end
</code></pre></div>

Note:

So we've covered input effects and how we can push them out of methods.
Output effects, unfortunately, aren't as easy to deal with.
It's obvious how to convert an input effect into an input value. 
It's less obvious how to convert an output effect into an output values.
We'll follow a similar strategy.


# The Command Pattern

Note:

The command pattern is a nice and convenient way to convert output effects into return values.
Let's refactor our trivial example code to use a Command.


# A Command

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class InsertFooResult
  attr_reader :x, :y, :z
  def initialize(x, y, z)
    @x = x
    @y = y
    @z = z
  end

  def eql?(other)
    self === other &&
      x == other.x && y == other.y && z == other.z
  end
end
</code></pre></div>

Note:

First, we need a way to encapsulate the arguments to the effect that we want to have.
In this case, we can create a read-only class that just carries those values around.

You really want these to be *values*, and not typical objects.
What's the difference between a value and an object, you ask?


# Values vs Objects

Note:

Are objects values? Not necessarily.
Objects have a notion of identity that is separate from the values of their member variables.
By default, objects in Ruby and most object oriented languages compare each other based on reference equality: these two objects are equal iff they refer to the same object in memory.
Two users, each with the same name and age, are different if they are stored in different places in memory.


# Objects

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class User
  attr_reader :name, :age

  def initialize(name, age)
    @name = name
    @age = age
  end
end

<span class="fragment">a = User.new("Matt", 28)
b = User.new("Matt", 28)</span>

<span class="fragment">a == b # False!</span>
</code></pre></div>

Note:

Here we've got a User class with name and age.
We instantiate two users with the same values.
These are different objects, and equality checking returns false for them.


# Values

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class User
  def eql?(other)
    name == other.name && age == other.age
  end
end
</code></pre></div>

Note:

Values, on the other hand, are equal if every component of the value is equal.
5 is equal to 5, regardless of where the two fives are stored in memory.
This modification to the User class converts it into a value, where two users are now equal if their name and age are the same.


# Values

[https://github.com/tcrayford/Values](https://github.com/tcrayford/Values)

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
User = Value.new(:name, :age)

user_1 = User.new("Matt", 28)
user_2 = User.new("Matt", 28)

user_1 == user_2
# true
</code></pre></div>

Note:

I'm going to refer to the `Values` library. The above code creates an immutable
value object with two fields, user and age. Equality is done by comparing the
members for equality.


## A value command

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
InsertFooResult = Value.new(:x, :y, :z)

cmd_1 = InsertFooResult.new(1, 2, 3)
cmd_2 = InsertFooResult.new(4, 3, 2)

puts cmd_1 == cmd_2
# false
</code></pre></div>

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
data InsertFooResult 
  = InsertFooResult Int Int Int
  deriving (Eq, Ord, Show)

cmd1 = InsertFooResult 1 2 3
cmd2 = InsertFuuResult 4 3 2

main = print (cmd1 == cmd2)
-- False
</code></pre></div>

Note:

This is that values library I talked about previously. Highly recommended. It's
even more concise than the Haskell!


# Using:

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class Foo
  def my_func(x, y, z)
    x + y + z, InsertFooResult.new(x, y, z)
  end
end

value, action = Foo.new.my_func(1, 2, 3)
</code></pre></div>

<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
myFunc 
  :: Int -> Int -> Int -> (Int, InsertFooResult)
myFunc x y z = 
  (x + y + z, InsertFooResult x y z)

(value, action) = myFunc 1 2 3
</code></pre></div>

Note:

Ok, so we're going to *return* the command value as one of the values returned from our method.

Ruby lets us return two things in a method, which is pretty cool.
It implicitly returns an array of stuff, which you can then destructure when you call the method.

Haskell uses a tuple type, which can be patern matched on in much the same way.


it's so easy

```java
class Pair<A, B> {
    public final A first;
    public final B second;

    public Pair(A a, B b) {
        this.first  = a;
        this.second = b;
    }
}
```

even java can do it

Note:

You can do this in less flexible languages, it's just not fun or convenient.
This is a Java implementation of a pair of values.


```java
class Foo {
    public Pair<Integer, InsertFooResult> 
    myFunc(int x, int y, int z) {
        return new Pair<Integer, InsertFooResult>(
            x + y + z,
            new InsertFooResult(x, y, z)
        );
    }
}
```

Note:

All languages can mimic this feature by returning a single composite value, so
don't fret if you're using Java or C# or PHP or whatever. Half of my dayjob is
in PHP and I use these techniques there!

Now we can write a test for it:


# Testing

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
describe Foo do
  it "does the thing" do
    expect(Foo.new.my_func(2, 3, 4))
      .to eq(
        <span class="fragment">[
          2 + 3 + 4,
          InsertFooResult.new(2, 3, 4)
        ]</span>
      )
  end
end
</code></pre></div>

Note:

We don't have to stub anything out. We're just comparing values to each other.
This is super easy, basic, 2 + 2 level stuff. Our output effect is now a simple
value.

I personally find this *much* nicer to look at than the previous code. We can
compare how much complexity we're invoking, and how much of that is difficult to
achieve in other languages.


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
describe "Foo" $ do
  prop "does the thing" $ \x y z ->
    myFunc x y z 
      \`shouldBe\` 
        (x + y + z, InsertFooResult x y z)
    
</code></pre></div>

Note:

And here's the test in Haskell. Easy to express as a QuickCheck property, too!
