# Methods and Functions

## the unit of

### uh, doing *stuff*

Note:

So, when we talk about "doing stuff" in computer programming, we have a bunch
of different ways of organizing it. If you're writing in OOP, you'll start by
making a class, and then defining some methods on it. In functional
programming, you start by defining a data type and writing functions that
operate on it. In imperative languages, you define procedures that run a
sequence of commands on the underlying machine.


# Values and Effects

# Input and Output

Note:

Let's talk about how we *use* methods and functions. Generally, we can talk
about a function in terms of the inputs and outputs that it has. We can also
talk about a function in terms of the values and effects that it deals with.


# Values

```ruby
class Foo
  def my_func(x, y)          # value!
    z = User.all.length
    FooResult.insert(x, y, z)
    x + y + z                # value!
  end
end
```

* Input arguments
* Return value

Note:

The *values* in this code snippet are the simple, easy bits. We pass in the
numbers x and y, which are both values. We return x + y + z, which is a simple
value.


# Effects

```ruby
class Foo
  def my_func(x, y)
    z = User.all.length       # effect!
    FooResult.insert(x, y, z) # effect!
    x + y + z
  end
end
```

* Input effects
* Output effects

Note:

The effects in this function are reading all of the users out of the database,
and inserting a new result into the database.


# Values and Effects


![](box.png)

Note:

Values are, at a first approximation, the things we pass directly into
functions or methods, and the things that are returned directly from functions.
Input effects are the things that provide information to the function that we
don't explicitly pass in. Output effects are the things that happen as a result
of calling the method, that aren't explicitly part of the return value.


# Values are *explicit*

# Effects are *implicit*

Note:

So values are explicit. Testing values is easy, and testing is an OK
approximation of good software. Effects are implicit. And testing effects is difficult.


# Testing Values:

```ruby
def add(x, y)
  x + y
end
```

```ruby
describe "add" do
  it "should add" do
    expect(add(2, 3)).to eq 5
  end
end
```

Note:

So, here's the super simple add function. Testing it is stupid easy. We just
pass in some input values, and make assertions about the output value. This
test is pretty silly, but it's easy to come up with more advanced test cases.


```ruby
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
```

Note:

Here, we're testing that add is associative and commutative. We're taking
a random sample of values, and ensuring that we can reorder our operations and
group them however we want. These tests are *easy* to write. They're fun,
almost. And they're kinda pretty! They look extremely close to the mathematical
definitions of associativity and commutativity.

This sounds trivial, but many difficult concepts in distributed systems involve
guaranteeing properties like commutativity. Making these properties easy to test
is important.


# Testing Effects:

```ruby
class Foo
  def my_func(x, y)
    z = User.all.length       # effect!
    FooResult.insert(x, y, z) # effect!
    x + y + z
  end
end
```

Note:

Testing effects is a lot harder. Suddenly we have to worry about what User is,
and what happens when we do FooResult. I'm going to evolve testing this example.


# Wrong

```ruby
describe "Foo#my_func" do
  it "adds inputs" do
    expect(Foo.new.my_func(1,2)).to eq(3)
  end
end
```

Note:

So this attempt is flat out wrong. However, on an uninitialized database with 0 users, it'll return the right answer. This is a *fragile* test, even though it may pass sometimes.


# Slow

```ruby
describe "Foo#my_func" do
  it "adds inputs" do
    User.insert(name: "Matt", age: 28)
    expect(User.all.length).to eq(1)
    expect(Foo.new.my_func(1,2)).to eq(4)
    x = FooResult.find_by(x: 1, y: 2, z: 1)
    expect(x).to_not be_nil
  end
end
```

Note:

So this isn't wrong anymore. However, the test relies on the database state, and
has to do five SQL queries in order to verify the code. These tests are
monstrously slow and will kill your TDD cycle, in addition to being fragile and
annoying to write.


# Stubs!

Note:

The next "level up" that often happens is to take advantage of stubs or mocks.
Let's look at that real quick:


```ruby
describe "Foo" do
  it "adds some numbers" do
    x, y, z = 3, 4, 3

    expect(User)
      .to receive(:all)
      .and_return([1,2,3])

    allow(FooResult)
      .to receive(:insert).with(x, y, z)

    expect(Foo.new.my_func(x, y))
      .to eq(x + y + z)
  end
end
```

Note:

This test is a lot nicer. We need to stub out the User.all method to ensure it
returns a value that suits our expectation. We also need to stub out the
FooResult class and verify that it receives the arguments we expect. Finally, we
can do some assertions on the actual values involved.

This kinda sucks! You can imagine extending this to more complex things, but it
gets even uglier, pretty quickly. Furthermore, stubs and mocks are pretty
controversial in the OOP community. They're not a clear best practice.


# Dependency Injection?

Note:

Dependency injection is usually heralded as the solution or improvement to just
stubbing out random global names. You define an interface (or duck type) for
what your objects need and pass them in your object initializer


```ruby
class Foo
  def initialize(user, foo_result)
    @user = user
    @foo_result = foo_result
  end

  def my_func(x, y)
    z = @user.all.length        # effect!
    result = x + y + z
    @foo_result.insert(x, y, z) # effect!
    result
  end
end
```

Note:

Dependency injection can be used to make testing like this a little easier,
especially in languages that aren't as flexible as Ruby. Instead of overriding a
global name, you make the class depend on a parameter that's local. Instead of
referring to the global User class, we're referring to the instance variable
user which is ostensibly the same thing. This is Good, as we've reduced the
coupling in our code, but we've introduced some significant extra complexity.
And the testing story isn't great, either:


```ruby
describe "Foo" do
  it "adds stuff" do
    x, y, user = 2, 3, double()
    user.stub(:all) { [1,2,3] }

    foo_result = double()
    foo_result.stub(:insert)

    expect(foo_result)
      .to receive(:insert).with(x, y, 3)

    foo = Foo.new(user, foo_result)

    expect(foo.my_func(x, y)).to eq(x + y + 3)
  end
end
```

Note:

This is clearly worse than before. If we're going by the metric that easier to
test code is better code, then this code *really* sucks. So dependency injection
is clearly not the obvious solution to this problem.

You can write helpers and stuff to obscure the difficulty of testing this. But
that doesn't make it *better*, it just hides the badness. Sometimes that's
great! Perfect is the enemy of good etc.


# Dependency Injection Is Still An Effect

Note:

Dependency injection feels like we're removing the effect. But we're not. We've
just moved it slightly. `self` in Ruby or `this` in Java, JS, PHP, etc. are
implicit parameters to object methods. This is something that Python
illuminates.


```python
# Python
class Foo(object):
    def __init__(self, user, foo_result):
        self.user = user
        self.foo_result = foo_result

    def my_func(self, x, y):
        z = self.user.all().length()
        self.foo_result.insert(x, y, z)
        return x + y + z
```

Note:

This is the Python code for our silly example. Where does `self` come from?
Well, it's attached to the object. So two identical looking calls to `my_func`
are going to return different results based on how the object was initialized.


# Not

### (obviously)

# Bad

Note:

The more volatile these parameters are, the more difficult they are to deal
with. If the `user` and `foo_result` inputs never change, then it's ezpz. If
they ever do change, though, then understanding `my_func` becomes a lot more
difficult.

In languages that can enforce that some members are final, then this is safer.
In languages that don't support immutability, you have to use discipline
instead.

In all of the above examples, we had to make our code and tests *significantly*
more complex. I want to make it simpler.


# Values vs Objects

Note:

Are objects values? Not necessarily. Objects have a notion of identity that is
separate from the values of their member variables. By default, objects in Ruby
and most object oriented languages compare each other based on reference
equality: these two objects are equal iff they refer to the same objet in
memory. Two users, each with the same name and age, are different if they are
stored in different places in memory.


# Objects

```ruby
class User
  attr_reader :name, :age
  def initialize(name, age)
    @name = name
    @age = age
  end
end

a = User.new("Matt", 28)
b = User.new("Matt", 28)

a == b # False!
```

Note:

Here we've got a User class with name and age. We instantiate two users with the
same values. THese are different objects, and equality checking returns false for them.


# Values

```ruby
class User
  # ...
  def eql?(other)
    name == other.name && age == other.age
  end
```

Note:

Values, on the other hand, are equal if every component of the value is equal. 5
is equal to 5, regardless of where the two fives are stored in memory. This
modification to the User class converts it into a value, where two users are now
equal if their name and age are the same.


# Values

[https://github.com/tcrayford/Values](https://github.com/tcrayford/Values)

```ruby
User = Value.new(:user, :age)
```

Note:

I'm going to refer to the `Values` library. The above code creates an immutable
value object with two fields, user and age. Equality is done by comparing the
members for equality
