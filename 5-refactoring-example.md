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
# Ruby
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

This method takes a user and a plan that we want to subscribe them to.  We're
going to iterate over all the users current subscriptions and cancel anything
on the same product.  Then we finally subscribe them to that plan.

This is the super imperative algorithm that requires stubs, mocks, etc. to
test. So we need to refactor it to have fewer effects, so the test story is
nicer!


```haskell
subcribe user plan = do
  for_ (userSubscriptions user) $ \sub ->
    when (subProduct sub == planProduct plan)
         (Stripe.cancel subscription)
  
  Stripe.subscribe user plan
```

Note:

So this is the same thing, but in Haskell. Haskell won't save you! You can
write some pretty nasty code in Haskell, and it's only a little more inconvenenient
than writing it in Ruby or PHP or whatever. 


# Issue Commands

## Get Money

Note:

Alright, so instead of that initial code, we're going to decouple the construction of our plan with the execution.


```ruby
# Ruby
Cancel    = Value.new(:subscription)
Subscribe = Value.new(:user, :plan)

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


```haskell
-- Haskell
data SubscribeCommand
    = Cancel Subscription
    | Subscribe User Plan
    deriving (Eq, Show)

subscribe :: User -> Plan -> [SubscribeCommand]
subscribe user plan = 
    cancels user ++ [Subscribe user plan]
  where
    cancels = map Cancel 
            . filter pred
            . userSubscriptions
    pred sub = subProduct sub == product
    product = planProduct plan
```

Note:

Here's the same thing in Haskell! We're defining a sum type for the various
commands that'll come out of the function, and we return a list of them. It's
just a pure function that maps a user and plan to a list of commands to
execute. This is a pure specification of our business logic.

An interesting thing here is that we're returning an array or list of commands
to execute.  In any case, it's now really easy to set this up and write tests:


```ruby
# Ruby
describe "subscribe" do
  it "should subscribe to an empty user" do
    user, plan = double(), double()
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


```haskell
-- Haskell
describe "Subscribe" $ do
  it "should subscribe to an empty user" $ do
    let 
      user = fakeUser { userSubscriptions = [] }
      plan = fakePlan { planProduct = "foo" }

    subscribe user plan 
      `shouldBe` 
        [Subscribe user plan] 
```

Note:

Here's that test in Haskell. It's basically the same thing, and a pretty clear
declaration of our business logic. If we have arbitrary instances for our
types, then we can do something similar:


```haskell
-- Haskell
describe "subscribe" $ do
  prop "should subscribe an empty user" $ 
    \user plan -> do
    let user' = user { userSubscriptions = [] }

    subscribe user' plan 
      `shouldBe` 
        [Subscribe user' plan]
```

Note:

Since our commands and inputs are just dumb data, we can easily generate
arbitrary values to get extra confidence that we're doing the right thing, as well as
ignoring stuff that's incidental to what we're working on.


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
verifying that our subscribe method cancels old plans on the same product.  The
test creates three stubs, makes the plan and subscription have the same
product, and finally makes some assertions about what the stubs contain.


# Discard Stubs, 

# Acquire Values

```ruby
# Ruby
User         = Value.new(:subscriptions)
Subscription = Value.new(:product)
Plan         = Value.new(:product, :amount)
```

Note:

OK, so if you're really anti-stubbing, then you hopefully are just using dumb
value classes. If you do that, then you don't need to stub anything out at all.
Here's our minimal data model.

After all, stubbing/mocking like this isn't easily generalizable to other
languages and environments. Value objects, on the other hand, are really easy
to implement in any language.


```ruby
describe "subscribe" do
  it "cancels a related plan" do
    sub  = Subscription.new("foo")
    user = User.new([sub])
    plan = Plan.new("foo", 123)
    
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

So here are the property tests for adding two numbers. It can kinda be tricky
to express business logic in such a mathematical way. Fortunately,
specifications for business logic often resemble these sorts of properties. We
can recover a more natural language specification for these properties as:


# Properties:

1. Subscribing a user to a plan should always issue a subscribe command for that plan
2. Subscribing a user to a plan should always issue cancel commands for subscriptions on the same product
3. Subscribing a user to a plan should not cancel plans on other products

Note:

These are the properties that we want to express in our business logic. And, as it happens, we can convert these directly to property tests!


```ruby
# Ruby
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
that the generator makes sensible choices: problems with QuickCheck's random
generation of values can certainly cause the test to not assert what you think
it asserts.


```haskell
describe "subscribe" $ do
  prop "it always subscribes to a given plan" $ 
    \user plan ->
      subscribe user plan 
        `shouldInclude`
          Subscribe user plan
```

Note:

The Haskell QuickCheck properties look pretty similar. While we don't have the
same correctness guarantees in Ruby and Haskell, you can tell that we're
getting pretty close.


```ruby
# Ruby
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

This property tests that we always cancel related plans.  We generate our
random user and plan, and then create the list of cancellations that we expect.
Finally, we assert that all of the cancellations exist in the commands array.


```haskell
describe "subscribe" $ do
  prop "it always cancels related plans" $ 
    \user plan -> do
      let 
        cancellations = 
          map Cancel 
            . subsOnProduct subs
            $ planProduct plan
        subs = userSubscriptions user

      subscribe user plan
        `shouldSatisfy` 
          (cancellations `isSubsetOf`)
```

Note:

And here we are in Haskell. This test works exactly the same logic as the above
Ruby code.


```ruby
# Ruby
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

Finally, we can test that we don't cancel any plans that aren't related to the
one we're subscribing for.  This technique is crazy powerful for verifying the
correctness of our business logic.


```haskell
describe "subscribe" $ do
  prop "doesn't cancel unrelated plans" $ 
    \user plan -> do
      let 
        unrelatedCancels =
          map Cancel
            . subsNotOnProduct subs
            $ planProduct plan
        subs = userSubscriptions user
  
      subscribe user plan 
        `shouldSatisfy`
          all (`notElem` unrelatedCancels)
```

Note:

I'm pretty pleased that our Ruby tests are about as good as our Haskell tests.
Where a lot of functional programming techniques feel awkward and clumsy when
ported to object oriented languages, this technique seems to work really well
for both.


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
