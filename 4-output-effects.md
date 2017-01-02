# Output Effects

```ruby
class Foo
  def my_func(x, y)
    z = User.all.length       
    FooResult.insert(x, y, z)  # output effect!
    x + y + z
  end
end
```

Note:

So we've covered input effects and how we can push them out of methods.
Output effects, unfortunately, aren't as easy to deal with.
We'll follow a similar strategy.
It's obvious how to convert an input effect into an input value.
It's less obvious how to convert an output effect into an output values.


# The Command Pattern

Note:

So the command pattern is a nice and convenient way to convert output effects into return values.
Let's refactor the above code to use a Command.


# A Command

```ruby
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


# A Shorthand

```ruby
InsertFooResult = Struct.new(:x, :y, :z)
```

Note:

Ruby structs are useful way to encapsulate this simple data in a lightweight
manner.


# Using:

```ruby
class Foo
  def my_func(x, y, z)
    x + y + z, InsertFooResult.new(x, y, z)
  end
end

value, action = Foo.new.my_func(1, 2, 3)
```

Note:

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

All languages can mimic this sort of effect by returning a single composite value.

Now we can write a test for it:


# Testing

```ruby
describe "Foo" do
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

We don't have to stub anything out.
We're just comparing values to each other.
This is super easy, basic, 2 + 2 level stuff.


# Doing commands

* Separate interpreter class
* An `interpret` method on the class itself

Note:

So, eventually you need to actually run your commands.
From a pure "separation of concerns" standpoint, I like having a separate class that interprets the effects.
You can also have an `interpret` method on the command class itself.


```ruby
class InsertFooResult
  def interpret
    FooResult.insert(x, y, z)
  end
end
```


```ruby
class InsertFooResultInterpreter
  def interpret(command)
    FooResult.insert(
      command.x, command.y, command.z
    )
  end
end
```
