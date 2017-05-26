# Refactoring Example

Note:

Ok, now I'm going to demonstrate a quick example of refactoring some code with commands.


# Business Logic

1. We're writing a subscription/billing system for our SaaS stuff
2. We have many products, and many plans for each product (small, large, enterprise)
3. We only want customers to have one plan for a given product at a time

Note:

Here's a brief overview of the spec we've been given.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
def subscribe(user, plan)
  user.subscriptions<span class="fragment">.each do |subscription|
    if subscription.product == plan.product
      subscription.cancel!
    end
  end</span>

  <span class="fragment">user.subscribe!(plan)</span>
end
</code></pre></div>

Note:

Let's look at the imperative formulation of this logic.
This method takes a user and a plan that we want to subscribe them to.
We're going to iterate over all the users current subscriptions and cancel anything on the same product.
Then we finally subscribe them to that plan.

This is the super imperative algorithm that requires stubs, mocks, etc. to
test. So we need to refactor it to have fewer effects, so the test story is
nicer!


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
subcribe user plan = do
  for_ (userSubscriptions user) $ \sub ->
    when (subProduct sub == planProduct plan)
         (Stripe.cancel subscription)
  
  Stripe.subscribe user plan
</code></pre></div>

Note:

So this is the same thing, but in Haskell. Haskell won't save you! You can
write some pretty nasty code in Haskell, and it's only a little more inconvenenient
than writing it in Ruby or PHP or whatever. 


# Issue Commands

## Get Money

Note:

Alright, so instead of that initial code, we're going to decouple the construction of our plan with the execution.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
Cancel    = Value.new(:subscription)
Subscribe = Value.new(:user, :plan)

def subscribe(user, plan)
  <span class="fragment">user.subscriptions</span><span class="fragment">.filter do |subscription|
    subscription.product == plan.product
  end</span><span class="fragment">.map do |subscription|
    Cancel.new(subscription)
  end</span><span class="fragment">.push(Subscribe.new(user, plan))</span>
end
</code></pre></div>

Note:

We start out by defining the Value commands that we need.

We start with the users current subscriptions. We filter that list, resulting in
a list of the users subscriptions that match the new plan's product. These are
the plans we want to cancel. We create a new Cancel command for each of these
subscriptions. Finally, we push a new Subscribe command to the end of that list,
subscribing them to the new plan.


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
data SubscribeCommand
    = Cancel Subscription
    | Subscribe User Plan
    deriving (Eq, Show)

<span class="fragment">subscribe :: User -> Plan -> [SubscribeCommand]
subscribe user plan = 
    cancels user ++ [Subscribe user plan]</span><span class="fragment">
  where
    cancels = map Cancel 
            . filter sameProduct
            . userSubscriptions
    sameProduct sub = subProduct sub == product
    product = planProduct plan</span>
</code></pre></div>

Note:

Here's the same thing in Haskell! We're defining a sum type for the various
commands that'll come out of the function, and we return a list of them. It's
just a pure function that maps a user and plan to a list of commands to
execute. This is a pure specification of our business logic.

An interesting thing here is that we're returning an array or list of commands
to execute.  In any case, it's now really easy to set this up and write tests:


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
describe "subscribe" do
  it "should subscribe to an empty user" do
    user = User.new(subscriptions: [])
    plan = Plan.new(product: "foo")

    commands = subscribe(user, plan)

    expect(commands)
      .to include(Subscribe.new(user, plan))

    expect(commands.length).to eq(1)
  end
end
</code></pre></div>

Note:

So our tests are pretty nice now. We don't have to worry about the details of
plan subscription. Perhaps that's handled through Stripe or some other
provider. We don't have to mock that API or anything.


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
describe "Subscribe" $ do
  it "should subscribe to an empty user" $ do
    let 
      user = fakeUser { userSubscriptions = [] }
      plan = fakePlan { planProduct = "foo" }

    subscribe user plan 
      \`shouldBe\` 
        [Subscribe user plan] 
</code></pre></div>

Note:

Here's that test in Haskell. It's basically the same thing, and a pretty clear
declaration of our business logic. 


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
describe "subscribe" do
  it "cancels a related plan" do
    old_sub = Subscription.new(product: "foo")
    user    = User.new(subscriptions: [old_sub])
    plan    = Plan.new(product: "foo")

    <span class="fragment">commands = subscribe(user, plan)</span>

    <span class="fragment">expect(commands)
      .to include(Subscribe.new(user, plan))</span>
    <span class="fragment">expect(commands)
      .to include(Cancel.new(old_sub))</span>
  end
end
</code></pre></div>

Note:

We can easily write tests for more complex business logic too.
Here we're going to verify that our subscribe method cancels old plans on the same product.
The test initializes our data so that the new plan and the old subscription have the same product.
Finally, we call our logic, and make some assertions about what the commands should contain.


# Property Testing

Note:

Remember adding? We wanted to verify properties about it, so we generated a
bunch of random values and asserted that the property held for all of the values
we generated.

We can describe properties of our business logic too. Then we can generate
random values and assert that they hold.


# Properties:

1. Subscribing a user to a plan should always issue a subscribe command for that plan
2. Subscribing a user to a plan should always issue cancel commands for subscriptions on the same product
3. Subscribing a user to a plan should not cancel plans on other products

Note:

These are the properties that we want to express in our business logic. And, as it happens, we can convert these directly to property tests!


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
describe "subscribe" do
  it "always subscribes to given plan" do
    forall_users do |user|
      forall_plans do |plan|
        <span class="fragment">expect(subscribe(user, plan))
          .to include(Subscribe.new(user, plan))</span>
      end
    end
  end
end
</code></pre></div>

Note:

So this property generates a bunch of random users, a bunch of random plans, and
asserts that the commands *always* include a subscribe command. We'll assume
that the generator makes sensible choices: problems with QuickCheck's random
generation of values can certainly cause the test to not assert what you think
it asserts.


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
describe "subscribe" $ do
  prop "it always subscribes to a given plan" $ 
    \user plan ->
      subscribe user plan 
        \`shouldInclude\`
          Subscribe user plan
</code></pre></div>

Note:

The Haskell QuickCheck properties look pretty similar. While we don't have the
same correctness guarantees in Ruby and Haskell, you can tell that we're
getting pretty close.


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
describe "subscribe" do
  it "always cancels related plans" do
    forall_users do |user| 
      forall_plans do |plan|
        <span class="fragment">cancellations = subs_on_product(
          user.subscriptions, plan.product
        ).map { |sub| Cancel.new(sub) }</span>
  
        <span class="fragment">expect(subscribe(user, plan))
          .to include(*cancellations)</span>
      end
    end
  end
end
</code></pre></div>

Note:

This property tests that we always cancel related plans.  We generate our
random user and plan, and then create the list of cancellations that we expect.
Finally, we assert that all of the cancellations exist in the commands array.


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
describe "subscribe" $ do
  prop "it always cancels related plans" $ 
    \user plan -> do
      let 
        cancellations = 
          <span class="fragment">map Cancel 
            . subsOnProduct (userSubscriptions user)
            $ planProduct plan</span>

      subscribe user plan
        \`shouldSatisfy\` 
          (\cmds -> cancellations \`isSubsetOf\` cmds)
</code></pre></div>

Note:

And here we are in Haskell.
This test works exactly the same logic as the above Ruby code.
I'm going to leave the "subscribing to a new plan should not cancel unrelated plans" logic to the audience. It's a fun exercise to play with.  

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
