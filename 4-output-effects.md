# Output Effects

```ruby
# Ruby
class Foo
  def my_func(x, y)
    z = User.all.length       
    FooResult.insert(x, y, z)  # output effect!
    x + y + z
  end
end
```

Note:

So we've covered input effects and how we can push them out of methods. Output
effects, unfortunately, aren't as easy to deal with. We'll follow a similar
strategy. It's obvious how to convert an input effect into an input value. It's
less obvious how to convert an output effect into an output values.


# The Command Pattern

Note:

So the command pattern is a nice and convenient way to convert output effects
into return values. Let's refactor the above code to use a Command.


# A Command

```ruby
# Ruby
class InsertFooResult
  attr_reader :x, :y, :z
  def initialize(x, y, z)
    @x = x
    @y = y
    @z = z
  end
end
```

Note:

First, we need a way to encapsulate the arguments to the effect that we want to
have. In this case, we can create a read-only class that just carries those
values around.

You want these to be values, because values are nice and easy. Making them a
class tempts us to add extra functionality.


# A value command

```ruby
# Ruby
InsertFooResult = Value.new(:x, :y, :z)
```

```haskell
-- Haskell
data InsertFooResult 
  = InsertFooResult Int Int Int
  deriving (Eq, Ord, Show)
```

Note:

This is that values library I talked about previously. Highly recommended. It's
even more concise than the Haskell!


# Using:

```ruby
# Ruby
class Foo
  def my_func(x, y, z)
    x + y + z, InsertFooResult.new(x, y, z)
  end
end

value, action = Foo.new.my_func(1, 2, 3)
```

```haskell
-- Haskell
myFunc x y z = 
  (x + y + z, InsertFooResult x y z)
```

Note:

Ok, so we're going to *return* the command value as one of the values returned from our method.

Ruby lets us return two things in a method, which is pretty cool.
It implicitly returns an array of stuff, which you can then destructure when you call the method.


```java
// Java
class Pair<A, B> {
    public final A first;
    public final B second;

    public Pair(A a, B b) {
        this.first  = a;
        this.second = b;
    }

    public static <A, B> 
    Pair<A, B> of(A a, B b) {
        return new Pair<A, B>(a, b); 
    }
}
```

Note:

You can do this in less flexible languages, it's just not fun or convenient.
This is a Java implementation of a pair of values.


```java
class Foo {
    public Pair<Integer, InsertFooResult> 
    myFunc(int x, int y, int z) {
        return Pair.of(
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

```ruby
# Ruby
describe Foo do
  it "does the thing" do
    expect(Foo.new.my_func(2, 3, 4))
      .to eq(
        [
          2 + 3 + 4,
          InsertFooResult.new(2, 3, 4)
        ]
      )
  end
end
```

Note:

We don't have to stub anything out. We're just comparing values to each other.
This is super easy, basic, 2 + 2 level stuff. Our output effect is now a simple
value.

I personally find this *much* nicer to look at than the previous code. We can
compare how much complexity we're invoking, and how much of that is difficult to
achieve in other languages.


```haskell
describe "Foo" $ do
  prop "does the thing" $ \x y z ->
    myFunc x y z 
      `shouldBe` 
        (x + y + z, InsertFooResult x y z)
    
```

Note:

And here's the test in Haskell. Easy to express as a QuickCheck property, too!
