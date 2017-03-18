# Refactoring Example

Note:

Ok, now I'm going to demonstrate a quick example of refactoring some code with commands.


# Business Logic

1. We're writing a subscription/billing system
2. We have many products, and many plans for each product (small, large, enterprise)
3. We only want customers to have one plan for a given product at a time

Note:

Here's a brief overview of the spec we've been given.


```ruby
def subscribe(user, plan)
  user.subscriptions.each do |subscription|
    if subscription.product == plan.product
      subscription.cancel!
    end
  end

  user.subscribe! plan
end
```

Note:

This method takes a user and a plan that we want to subscribe them to.
We're going to iterate over all the users current subscriptions and cancel anything on the same product.
Then we finally subscribe them to that plan.


```ruby
def subs_on_product(subscriptions, product)
  subscriptions.filter do |subscription|
    subscription.product == product
  end
end

def subscribe(user, plan)
  subs_on_product(
    user.subscriptions, plan.product
  ).each(&:cancel!)
  
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
Cancel    = Value.new(:subscription)
Subscribe = Value.new(:user, :product)

def subscribe(user, plan)
  user.subscriptions.filter do |subscription|
    subscription.product == plan.product
  end.map do |subscription|
    Cancel.new(subscription)
  end.push(Subscribe.new(user, plan))
end
```

Note:

We start with the users current subscriptions. We filter that list, resulting in
a list of the users subscriptions that match the new plan's product. These are
the plans we want to cancel. We create a new Cancel command for each of these
subscriptions. Finally, we push a new Subscribe command to the end of that list,
subscribing them to the new plan.

An interesting thing here is that we're returning an array of commands to execute.
In any case, it's now really easy to set this up and write tests:


```ruby
describe "subscribe" do
  it "should subscribe to an empty user" do
    user, plan = 2.times { double() }
    user.stub(:subscriptions) { [] }
    plan.stub(:product) { "foo" }

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

We *are* stubbing out objects here. In this sense, though, it is less that we
are stubbing methods on global terms, and more that we are documenting the duck
type that our method works with. Smaller duck types are easier to reuse and
compose, so the less stubbing and test setup we have to do, the better.


```ruby
describe "subscribe" do
  it "cancels a related plan" do
    user, plan, old_sub = 3.times { double() }
    old_sub.stub(:product) { "foo" }
    plan.stub(:product) { "foo" }
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

We can easily write tests for more complex business logic too. Here we're
verifying that our subscribe method cancels old plans on the same family.


# Discard Stubs, 

# Acquire Values

```ruby
User         = Value.new(:subscriptions)
Subscription = Value.new(:product)
Plan         = Value.new(:product)
```

Note:

OK, so if you're really anti-stubbing, then you hopefully are just using dumb value classes. If you do that, then you don't need to stub anything out at all. Here's our minimal data model.


```ruby
describe "subscribe" do
  it "cancels a related plan" do
    sub  = Subscription.new("foo")
    user = User.new([sub])
    plan = Plan.new("foo")
    
    commands = subscribe user, plan
    
    expect(commands)
      .to include(Subscribe.new(user, plan))
    expect(commands)
      .to include(Cancel.new(sub))
    expect(commands.length).to eq(2)
  end
end
```

Note:

And here are our fancy new stub-free tests.


# Property Testing

Note:

Remember adding? We wanted to verify properties about it, so we generated a
bunch of random values and asserted that the property held for all of the values
we generated.

We can describe properties of our business logic too. Then we can generate
random values and assert that they hold.


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

So here are the property tests for adding two numbers. We can recover a more natural language specification for these properties as:


# Properties:

1. Subscribing a user to a plan should always issue a subscribe command for that plan
2. Subscribing a user to a plan should always issue cancel commands for subscriptions on the same product
3. Subscribing a user to a plans should not alter plans on other products


```ruby
describe "subscribe" do
  it "always subscribes to given plan" do
    forall_users_and_plans do |user, plan|
      expect(subscribe(user, plan))
        .to include(Subscribe.new(user, plan))
    end
  end
end
```

Note:

So this property generates a bunch of random users, a bunch of random plans, and
asserts that the commands *always* include a subscribe command. We'll assume
that the generator makes sensible choices.


```ruby
describe "subscribe" do
  it "always cancels related plans" do
    forall_users_and_plans do |user, plan|
      cancellations = subs_on_product(
        user.subscriptions, plan.product
      ).map { |sub| Cancel.new sub }

      expect(subscribe(user, plan))
        .to include(*cancellations)
    end
  end
end
```

Note:

This property tests that we always cancel related plans.


```ruby
describe "subscribe" do
  it "doesn't cancel unrelated plans" do
    forall_users_and_plans do |user, plan|
      unrelated_cancels = subs_not_on_product(
        user.subscriptions, plan.product
      ).map { |s| Cancel.new(s) }

      expect(subscribe(user, plan))
        .to not_include(*unrelated_cancels)
    end
  end
end
```

Note:

Finally, we can test that we don't cancel any plans that aren't related to the one we're subscribing for.
This technique is crazy powerful for verifying the correctness of our business logic.


# Property Testing

- [QuickCheck](https://hackage.haskell.org/package/QuickCheck) (Haskell)
- [QuickCheck](https://www.google.com/search?q=quickcheck+github) (for lots of other languages!)
- [Propr](https://github.com/kputnam/propr): Ruby library

Note:

QuickCheck is the original property testing library, written in Haskell. There's
an interesting paper on how it's used that I'd recommend reading. QuickCheck has
been ported to many other langauges, though it's somewhat less pleasant because
you need to specify generators explicitly and many langauges don't encourage
working with easily testable values in the same way that Haskell does.
