# Input Effects

# $\to$

# Input Value

Note:

Maybe we can turn our effects into values?


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
class Foo
  def my_func(x, y)         
    z = User.all.length
    FooResult.insert(x, y, z)
    x + y + z                
  end
end
</code></pre></div>

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


<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
# Old:
class Foo
  def my_func(x, y)         
    z = <b><i>User.all.length</i></b>       # effects :(
    FooResult.insert(x, y, z)
    x + y + z                
  end
end

<span class="fragment"># New:
class Foo
  def my_func(x, y, z)
    FooResult.insert(x, y, z)
    x + y + z
  end
end</span>
</code></pre></div>

Note:

In this example, we've stopped talking to the database, and now just take the
length of all the users as a parameter.


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
myFunc :: Int -> Int -> Int -> IO Int
myFunc x y z = do
    insert (FooResult x y z)
    pure (x + y + z)
</code></pre></div>

Note:

And here's the Haskell. It's pretty much exactly what you'd expect.
Let's look at our testing story now.


# Testing:

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
describe Foo do
  it "adds stuff" do
    x, y, z = 1, 2, 3

    <span class="fragment">allow(FooResult)
      .to receive(:insert)
      .with(x, y, z)</span>

    expect(Foo.new.my_func(x, y, z)).to eq 6
  end
end
</code></pre></div>

Note:

Here's our test for this.
We have verified now that we get the correct value output for a given value input.

We still need to use a stub to ensure that we get the right *effect* output though.


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
describe "Foo" $ do
  prop "adds stuff" $ \x y z ->
    Foo.fooResult x y z 
      \`shouldReturn\`
        x + y + z
</code></pre></div>

Note:

The Haskell version even allows us to turn this into a QuickCheck property test
now, which gives us way more confidence on the correctness of this
implementation. Here, we're generating a bunch of random values for x, y, and
arr, and we're asserting that the property holds for these values.


# Input effects are ezpz

<div id="lang-logo"> <img src="1000px-Ruby_logo.svg.png" id="lang"/> <pre><code class="lang-ruby hljs" data-trim data-noescape>
def should_bill?(user_id)
  user = User.find(user_id)
  user.last_billing_date <= Time.now - 30.days
end
</code></pre></div>

Note:

Pop quiz!  
Does this function have any input effects? If so, what are they?


<div id="lang-logo"><img src="haskell_logo.svg" id="lang"/><pre><code class="lang-haskell hljs" data-trim data-noescape>
shouldBill :: UserId -> IO Bool
shouldBill userId = do
  user <- Sql.get userId
  time <- getCurrentTime
  pure $
    lastBillingDate user 
      \`isBefore\` 
        time .-^ days 30
</code></pre></div>

Note:

Haskell makes it super easy to figure out whenever you have input effects. The
IO type is basically "I do all the effects," and if you're binding out of IO or
anything like IO, then you've got input effects. This code kinda cheats the
answer for us: the user is an input effect, as is getting the current time.
