# Refactoring Example

Note:

Ok, now I'm going to demonstrate a quick example of refactoring some code with commands.


```ruby
def subscribe(user, plan)
  user.subscriptions.each do |subscription|
    if subscription.family == plan.family
      subscription.cancel!
    end
  end

  user.subscribe! plan
end
```

Note:

This method takes a user and a plan that we want to subscribe them to.
Plans have families, so you can have small/medium/large plans on the same overall product.
We're going to iterate over all the users current subscriptions and cancel anything on the same family.
Then we finally subscribe them to that plan.


```ruby
def subscriptions_to_cancel_for(user, family)
  user.subscriptions.filter do |subscription|
    subscription.family == family
  end
end

def subscribe(user, plan)
  subscriptions_to_cancel_for(user, plan.family)
    .each(&:cancel!)
  
  user.subscribe! plan
end
```

Note:

Ok, so commands are a heavier weight technique that don't work too well on tiny examples like this.
For this bit of code, it's easy enough to factor out the values entirely.
Now we have  a pure function in `subscriptions_to_cancel_for`, and our `subscribe` method is 100% effectful.
The effects are isolated and that's great.
But let's try it again with commands.


```ruby
Cancel = Struct.new(:subscription)
Subscribe = Struct.new(:user, :product)

def subscribe(user, plan)
  result = []

  user.subscriptions.each do |subscription|
    if subscription.family == plan.family
      result << Cancel.new subscription
    end
  end

  result << Subscribe.new user, plan
  result
end
```

Note:

explain the code snippet
An interesting thing here is that we're returning an array of commands to execute.
In any case, it's now really easy to set this up and write tests:


```ruby
Cancel    = Struct.new(:subscription)
Subscribe = Struct.new(:user, :product)

def subscribe(user, plan)
  user.subscriptions.filter do |subscription|
    subscription.family == plan.family
  end.map do |subscription|
    Cancel.new(subscription)
  end.push(Subscribe.new(user, plan))
end
```

Note:

This is a more functional implementation. We select the users subscriptions
that are in the plan's family, map over them with the Cancel action, and push
the Subscribe action to the end of the array.


```ruby
describe "subscribe" do
  it "should subscribe to an empty user" do
    user, plan = 2.times { double() }
    user.stub(:subscriptions) { [] }
    plan.stub(:family) { "foo" }

    commands = subscribe user, plan

    expect(commands)
      .to include(Subscribe.new(user, plan))

    expect(commands.length).to eq(1)
  end
end
```

Note:

So our tests are pretty nice now. We don't have to worry about the details of
plan subscription. Perhaps that's handled through Stripe or some other
provider. We don't have to mock that API or anything.


```ruby
describe "subscribe" do
  it "cancels a related plan" do
    user, plan, old_sub = 3.times { double() }
    old_sub.stub(:family) { "foo" }
    plan.stub(:family) { "foo" }
    user.stub(:subscriptions) { [old_sub] }
    commands = subscribe user, plan

    expect(commands)
      .to include(Subscribe.new(user, plan))
    expect(commands)
      .to include(Cancel.new(old_sub))
    expect(commands.length).to eq(2)
  end
end
```

Note:

We can easily write tests for more complex business logic too.
You may notice there are a lot of stubs going on still.
The stubs are just the easiest way to get this stuff on a slide. If the classes in question are just a value objects, that is, no methods, just immutable data, then we don't need to stub.


```ruby
User         = Struct.new(:subscriptions)
Plan         = Struct.new(:family)
Subscription = Struct.new(:family)

describe "subscribe" do
  it "same thing" do
    plan     = Plan.new("foo")
    old_sub  = Subscription.new("foo")
    user     = User.new([old_sub])
    commands = subscribe(user, plan)
    expect(commands)
      .to include(Subscribe.new(user, plan))
    expect(commands)
      .to include(Cancel.new(old_sub))
    expect(commands.length).to eq(2)
  end
end
```

Note:

This is the same test, but using value objects instead. Setup is a little
easier and cleaner.
