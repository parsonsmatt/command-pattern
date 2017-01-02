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
    FooResult.insert(x, y, z)  # effect!
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
almost. And they're kinda pretty!


# Testing Effects:

```ruby
class Foo
  def my_func(x, y)
    z = User.all.length        # effect!
    FooResult.insert(x, y, z)  # effect!
    x + y + z
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
    x, y, z = 3, 4, 3

    expect(User)
      .to receive(:all)
      .and_return((1..z).to_a)

    allow(FooResult)
      .to receive(:insert).with(x, y, z)

    expect(Foo.new.my_func(x, y)).to eq(x + y + z)
  end
end
```

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
    foo_result.insert(x, y, z)  # effect!
    result
  end
end
```

Note:

Dependency injection can be used to make testing like this a little easier,
especially in languages that aren't as flexible as Ruby. Instead of overriding a global name, you make the class depend on a parameter that's local. Instead of referring to the global User class, we're referring to the instance variable user which is ostensibly the same thing. This is Good, as we've reduced the coupling in our code, but we've introduced some significant extra complexity. And the testing story isn't great, either:


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

This is clearly worse than before. If we're going by the metric that easier to test code is better code, then this code *really* sucks. So dependency injection is clearly not the obvious solution to this problem.

You can write helpers and stuff to obscure the difficulty of testing this.
But that doesn't make it *better*, it just hides the badness.
Sometimes that's great!
Perfect is the enemy of good etc.


# Dependency Injection Is Still An Effect

Note:

Dependency injection feels like we're removing the effect.
But we're not. We've just moved it slightly.
`self` in Ruby or `this` in Java, JS, PHP, etc. are implicit parameters to object methods.
This is something that Python illuminates.


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


# Dependency Injection,

## Functionally

Note:

As a last aside, you can do dependency injection by passing in
functions/methods directly to the method you're calling. This is a powerful
technique which makes code much more flexible.


```ruby
def my_func(x, y, z, foo_result)
  foo_result.call(x, y, z)
  x + y + z
end

it "my_func" do
  called = false
  f = -> (_, _, _) { called = true }
  expect(my_func(x, y, f)).to eq(6)
  expect(called).to be_true
end
```

Note:

There's still an effect in this code. We're non-locally modifying a reference.
Which is kind of gross. We can get around that by returning two values: the
result of calling `foo_result` and the actual return value.


```ruby
def my_func(x, y, z, foo_result)
  x + y + z, foo_result.call(x, y, z)
end

it "my_func" do
  f = -> (_, _, _) { true }
  expect(my_func(x, y, f)).to eq([6, true])
end
```

Note:

We're back to two easy tests. Our function just returns true that it is called,
and we test value equality. Nice!


# Aside:

## Resource Management

Note:

This pattern is also *great* for automatic resource management. 


```ruby
def with_database(connection_info, callback)
  result = nil

  begin
    connection = Connection.new connection_info
    result = callback.call connection
  ensure
    connection.close 
  end

  result
end
```

Note:

This function accepts the database connection information and a function that
uses the database connection, finally returning a value of some generic type
`A`. The closing of the connection is handled by this function, so that we
can't leak any resources.


```ruby
def do_work
  with_database("connectpls", -> (conn) do
    count = conn.execute "
      SELECT count(*) FROM users
    " 

    raise UnpopularAppException if count < 15

    puts "hooray we made it"
    count
  end)
end
```

Note:

So we open a database connection and execute the count query.
If the count is less than 15, we raise an exception.
Otherwise, we return the count and print a joyous message.
The `with_database` function is handling the resources for us, so we can't
forget to close or misuse the connection.


```haskell
withDatabase :: String -> (Conn -> IO a) -> IO a
withDatabase info = 
    bracket (newConnection info) closeConnection

doWork :: IO Int
doWork =
  withDatabase "connectpls" (\conn -> do
    count <- execute conn 
      "SELECT count(*) FROM users"
    when (count < 15) 
      (throwIO UnpopularAppException)
    putStrLn "hooray we made it"
    pure count
  )
```
