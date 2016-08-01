*picture: pipelines in parallel*

Note:

So categories are the essence of composition.
But we often want to adapt different forms of composition to each other.


*picture of set with a->b function*

Note:

Also, what's up with sets having functions between sets,


*picture of category with endomorphism*

Note:

And categories only having arrows to the same objects?
How do we map categories to categories, like functions map sets to sets?


These are problems to solve!

How to solve them?


*picture of two categories*

Note:

So, first, in order to make this mapping, we'll need two categories.
We'll call them C and D.


*picture of arrow between the two*

Note:

And I guess we'll need an arrow between them, we'll call it F.
We want F to play nicely with our intuition on what makes categories important: identity and composition.
The first thing it does is map objects to objects.


*picture of mapping an object to an object*

Note:

Next, we need to map *arrows* to arrows!


*picture of mapping an arrow to an arrow*

Note:

So how do we choose which arrows to map to?
It seems wrong to map an identity arrow in C to something other than an identity arrow in D.
So identity arrows in C must map to identity arrows in D.


*diagram of identity arrow mapping*

Note:

So, if we're preserving properties on these categories, then we also want to preserve composition.
We can use these diagrams of objects and arrows to pretty clearly talk about these things.
The *rule* for these diagrams, informally, is that we should be able to follow the arrows, and it doesn't matter *how* we get somewhere, just that some chain exists between our start and destination.


*diagram of functor composition law*

Note:

So we can follow the arrows here -- the composition law means that it can't matter whether we map compositions or compose mappings.


# Fun with Functors

Note:

We've just described a functor.


# Formally,

A functor $F$ maps a category $\mathbb{C}$ to a category $\mathbb{D}$ (often referred to as $F(\mathbb{C})$), such that:

$$F(id_C(a)) = id_D(F(a))$$

$$F(f) \circ F(g) = F(f \circ g)$$


# Lets start with simple categories and functors between them

Note:

Remember how all monoids are categories?
We can get a sense for how functors work by looking at these very simple categories.


# String to Sum

Composing strings: appending them together

Composing numbers: adding them together

A functor between them, then, is a function `String -> Int` that respects their respective compositions.


It must be some function `f` such that:

$$f(id_S(a)) = id_I(f(a))$$

$$f(a ++ b) = f(a) + f(b)$$

Note:

The string identity applied beforehand and the integer identity applied afterwards must agree.
And the string appending before hand must agree with the integer summing after.


