# Methods and Functions

## the unit of

### uh, doing *stuff*

Note:

So, when we talk about "doing stuff" in computer programming, we have a bunch
of different ways of organizing it. If you're writing in OOP, you'll start by
making a class, and then defining some methods on it. In functional
programming, you start by defining a data type and writing functions that
operate on it.


# Values and Effects

# Input and Output

Note:

Let's talk about how we *use* methods and functions. Generally, we can talk
about a function in terms of the values and effects that it has. We can also
talk about a function in terms of the values and effects that it has.


# Values

```ruby
class Foo
  def my_func(x, y)          # value!
    z = User.all.length
    result = x + y + z
    FooResult.insert x, y, z
    return result            # value!
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
    result = x + y + z
    FooResult.insert x, y, z  # effect!
    return result
  end
end
```

* Input effects
* Output effects

Note:

The effects in this function are reading all of the users out of the database,
and inserting a new result into the database.


# Values and Effects

*TODO: drawing of box with input/outputs*

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
def add x, y
  x + y
end
```

```ruby
describe "add" do
  it "should add" do
    expect(add 2, 3).to eq 5
  end
end
```

Note:

So, here's the super simple add function. Testing it is stupid easy. We just
pass in some input values, and make assertions about the output value. This
test is pretty silly, but it's easy to come up with more advanced test cases.


# Testing Values:

```ruby
describe "Add" do
  it "is commutative" do
    100.times do
      x = Random::rand
      y = Random::rand
      expect(x + y).to eq(y + x)
    end
  end

  it "is associative" do
    100.times do
      x = Random::rand
      y = Random::rand
      z = Random::rand
      expect((x + y) + z).to eq(x + (y + z))
    end
  end
end 
```

Note:

Here, we're testing that add is associative and commutative. We're taking
a random sample of values, and ensuring that we can reorder our operations and
group them however we want. These tests are *easy* to write. They're fun,
almost. And they're kinda pretty!


# Testing Effects:

```ruby
class Foo
  def my_func(x, y)
    z = User.all.length       # effect!
    result = x + y + z
    FooResult.insert x, y, z  # effect!
    return result
  end
end
```

ugh

Note:

Testing effects is a lot harder. Suddenly we have to worry about what User is, and what happens when we do FooResult.


# Testing Effects:

```ruby
describe "Foo" do
  it "adds some numbers" do
    x = 3
    y = 4
    z = 3
    expect(User).to receive(:all).and_return (1..z).to_a
    expect(FooResult).to receive(:insert).with(x, y, z)
    expect(Foo.my_func x, y).to eq(x + y + z)
  end
end
```

ugh!

Note:

Here's our test. We need to stub out the User.all method to ensure it returns
a value that suits our expectation. We also need to stub out the FooResult
class and verify that it receives the arguments we expect. Finally, we can do
some assertions on the actual values involved.

This kinda sucks! You can imagine extending this to more complex things, but it
gets even uglier, pretty quickly. Furthermore, stubs and mocks are pretty
controversial in the OOP community. They're not a clear best practice.


# Dependency Injection?

```ruby
class Foo
  attr_reader :user, :foo_result

  def initialize(user, foo_result)
    @user = user
    @foo_result = foo_result
  end

  def my_func(x, y)
    z = user.all.length        # effect!
    result = x + y + z
    foo_result.insert x, y, z  # effect!
    result
  end
end
```

Note:

Dependency injection can be used to make testing like this a little easier,
especially in languages that aren't as flexible as Ruby. Instead of overriding a global name, you make the class depend on a parameter that's local. Instead of referring to the global User class, we're referring to the instance variable user which is ostensibly the same thing. This is Good, as we've reduced the coupling in our code, but we've introduced some significant extra complexity. And the testing story isn't great, either:


# Dependency Injection...

```ruby
describe "Foo" do
  it "adds stuff" do
    x, y = 2, 3
    user = double()
    user.stub(:all) { [1,2,3] }

    foo_result = double()
    foo_result.stub(:insert)

    expect(foo_result).to receive(:insert).with(x, y, 3)

    foo = Foo.new(user, foo_result)

    expect(foo.my_func(x, y)).to eq(x + y + 3)
  end
end
```

Note:

This is clearly worse than before. If we're going by the metric that easier to test code is better code, then this code *really* sucks. So dependency injection is clearly not the obvious solution to this problem.
