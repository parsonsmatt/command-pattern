# Resolving Input Effects


```ruby
# Ruby
class Foo
  def my_func(x, y)         
    z = User.all.length
    FooResult.insert x, y, z
    x + y + z                
  end
end
```

Note:

Ok, so let's look at our example again. We want to get rid of that input effect
so we can make the method easier to test, and by extension, easier to understand,
modify, reuse, etc.


# three magic steps

1. Identify input effects
<!-- .element: class="fragment" -->
2. Take result of input effect as a parameter
<!-- .element: class="fragment" -->
3. High five your coworkers
<!-- .element: class="fragment" -->


```ruby
# Ruby
class Foo
  def my_func(x, y, users)
    z = users.length
    FooResult.insert(x, y, z)
    x + y + z
  end
end
```

Note:

Fortunately, resolving input effects is super easy. You identify the input
effects, and then take them as a parameter. In this example, we've stopped
talking to the database, and now just take the object as a parameter. The tests
just got *way* simpler.


```haskell
-- Haskell
myFunc x y users = do
    let z = length users
    insert (FooResult x y z)
    pure (x + y + z)
```

Note:

And here's the Haskell. It's pretty much exactly what you'd expect.


# Testing:

```ruby
# Ruby
describe Foo do
  it "adds stuff" do
    x, y, arr = 1, 2, [1,2,3]

    allow(FooResult)
      .to receive(:insert)
      .with(x, y, arr.length)

    expect(Foo.new.my_func(x, y, arr)).to eq 6
  end
end
```

Note:

Three lines of test code. One of those is just initialize values. Nice!


```haskell
-- Haskell
describe "Foo" $ do
  prop "adds stuff" $ \arr x y ->
    Foo.fooResult x y arr 
      `shouldReturn`
        x + y + length arr
```

Note:

The Haskell version even allows us to turn this into a QuickCheck property test
now, which gives us way more confidence on the correctness. of this
implementation.


# Draw the input line?


Why not...?

```ruby
# Ruby
class Foo
  def my_func(x, y, z)
    FooResult.insert x, y, z
    x + y + z
  end
end

describe Foo do
  it "uhh" do
    x, y, z = 1, 2, 3
    allow(FooResult)
      .to receive(:insert).with(x, y, z)
    expect(Foo.new.my_func(x, y, z)).to eq 6
  end
end
```

Note:

You might know about the law of Demeter, which is roughly is described as: the
fewer expectations you have on your inputs, the easier your code is to deal
with, and the less likely it is to be fragile. In this example, we just pass in
the length of the users collection directly, removing that dependency entirely.

The code and the test fit on the same slide now! That's new.


# Input effects are ezpz

```ruby
# Ruby
def should_bill?(user_id)
  user = User.find(user_id)
  user.last_billing_date <= Time.now - 30.days
end
```

Note:

It's easy to resolve input effects. You just take the value you'd extract as a
parameter.  Does this function have any input effects? If so, what are they?


```haskell
-- Haskell
shouldBill userId = do
  user <- Sql.get userId
  time <- getCurrentTime
  pure (
    lastBillingDate user 
      `isBefore` 
        time .-^ days 30
    )
```

Note:

Haskell makes it super easy to figure out whenever you have input effects. The
IO type is basically "I do all the effects," and if you're binding out of IO or
anything like IO, then you've got input effects. This code kinda cheats the
answer for us: the user is an input effect, as is getting the current time.


# Input effects are ezpz

```ruby
# Ruby
def should_bill?(user, time)
  user.last_billing_date <= time - 30.days
end
```

```haskell
-- Haskell
shouldBill user time =
  lastBillingDate user `isBefore` time .-^ days 30
```

Note:

We've extracted all of the input effects from the method. The method depends
solely on the values we pass in as input now. The Ruby starts to look a lot
like the Haskell!


# Comparison of tests:

Note:

OK, so let's compare the tests of our methods, before and after extracting the
input effects.


## Implicit:

```ruby
# Ruby
it "is true if older than 30 days ago" do
  user = double()
  user.stub(:last_billing_date) { 31.days.ago }

  expect(User)
    .to receive(:find).with(1)
    .and_return(user) 

  expect(should_bill?(1)).to be_true
end 
```

Note:

We're overriding the global User name and making it return a stub. Since we
can't even do this in Haskell I won't show the test, and because (let's be
real) making our functions abstract in the functionality they do sounds boring.


## Explicit:

```ruby
# Ruby
it "is false if newer than 30 days" do
  user = double()
  user.stub(:last_billing_date) { 15.days.ago }

  expect(should_bill?(user, Time.now))
    .to be_false
end
```

Note:

These tests are pretty similar. Now we get to explicitly control the time that
we're comparing against. We also don't have to worry about mocking the User
class. We just create some object that responds to last billing date, and can
provide our constraints.
