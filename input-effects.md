# Resolving Input Effects


```ruby
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


# Testing:

```ruby
describe Foo do
  it "adds stuff" do
    x, y, arr = 1, 2, [1,2,3]
    expect(FooResult).to receive(:insert).with(x, y, arr.length)
    expect(Foo.new.my_func(x, y, arr)).to eq 6
  end
end
```

Note:

Three lines of test code. One of those is just initialize values. Nice!


# Draw the input line?


Why not...?

```ruby
class Foo
  def my_func x, y, z
    FooResult.insert x, y, z
    x + y + z
  end
end

describe Foo do
  it "uhh" do
    x, y, z = 1, 2, 3
    expect(FooResult).to receive(:insert).with(x, y, z)
    expext(Foo.new.my_func x, y, z).to eq 6
  end
end
```

Note:

Recommendation of Demeter. You might recognize the law of Demeter. The fewer
expectations you have on your inputs, the easier your code is to deal with, and
the less likely it is to be fragile. In this example, we just pass in the
length of the users collection directly.


# Input effects are ezpz

```ruby
def should_bill? user_id
  user = User.find user_id
  user.last_billing_date >= Time.now - 30.days
end
```

Note:

It's easy to resolve input effects. You just take the value you'd extract as a parameter.
Does this function have any input effects? If so, what are they?


# Input effects are ezpz

```ruby
def should_bill? user, time
  user.last_billing_date >= time - 30.days
end
```

Note:

We've extracted all of the input effects from the method. The method depends solely on the values we pass in as input now.


# Comparison of tests:

## Implicit:

```ruby
it "is true if older than 30 days ago" do
  user = double()
  user.stub(:last_billing_date) { 31.days.ago }
  expect(User).to receive(:find).with(1).and_return(user) 
  expect(should_bill? 1).to eq true
end 
```


# Comparison of tests:

## Explicit:

```ruby
it "is false if newer than 30 days" do
  user = double()
  user.stub(:last_billing_date) { 15.days.ago }
  expect(should_bill? user, Time.now).to eq false
end
```

Note:

These tests are pretty similar. Now we get to explicitly control the time that
we're comparing against. We also don't have to worry about stubbing anything.


# One step farther

```ruby
def should_bill? billing_date, time
  billing_date >= time - 30.days
end

it "does the right thing" do
  expect(should_bill? 15.days.ago, Time.now)
    .to eq false
end
```
