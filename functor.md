# Fun with Functors


# Formally,

A functor $F$ maps a category $\mathbb{C}$ to a category $\mathbb{D}$ (often referred to as $F(\mathbb{C})$), such that:

$$F(id_C(a)) = id_D(F(a))$$

$$F(f) \circ F(g) = F(f \circ g)$$


<!-- .slide: data-background-size="contain" data-background="./altfunctor.jpg" -->

Note:

So, first, in order to make this mapping, we'll need two categories.
We'll call them C and D.
We want F to play nicely with our intuition on what makes categories important: identity and composition.
The first thing it does is map objects to objects.
Next, we need to map *arrows* to arrows!
So how do we choose which arrows to map to?
It seems wrong to map an identity arrow in C to something other than an identity arrow in D.
So identity arrows in C must map to identity arrows in D.

So, if we're preserving properties on these categories, then we also want to preserve composition.
We can use these diagrams of objects and arrows to pretty clearly talk about these things.
The *rule* for these diagrams, informally, is that we should be able to follow the arrows, and it doesn't matter *how* we get somewhere, just that some chain exists between our start and destination.


## it can't matter 

## whether we map compositions 
<!-- .element: class="fragment" -->

## or compose mappings
<!-- .element: class="fragment" -->


# Start simple

Note:

Remember how all monoids are categories?
We can get a sense for how functors work by looking at these very simple categories.


# String to Sum

Composing strings: appending them together

Composing numbers: adding them together

A functor between them, then, is a function `String -> Int` that respects their respective compositions.


It must be some function $f$ such that:

$$f(id_S(a)) = id_I(f(a))$$

$$f(a +_S b) = f(a) +_I f(b)$$

Note:

The string identity applied beforehand and the integer identity applied afterwards must agree.
And the string appending before hand must agree with the integer summing after.


One candidate: `length`!

```ruby
# Python
def id(x):
    return x

len(id("hello"))  == id(len("hello"))

len("he" + "llo") == len("he") + len("llo")
```

Note:

Length does this right. It maps identity to identity and composition to composition, so length is a functor from the category of strings to the category of sums.


# Functors in Code

Note:

So what does a functor look like in code?
Generally, we've already lifted the objects in our program. This tends to be
pretty simple. We have to provide a means of taking an ordinary function and
mapping it over our functor -- this is the mapping of morphisms.


# Ruby

```ruby
module Functor
  def fmap f
    map f
  end
end

class Maybe
  include Functor
  ...
end
```

Note:

In Ruby, Python, and other dynamic languages, functors aren't all that special.
It's just anything you can call map on, and since you can call whatever you
want on whatever you want (yay duck types!), you don't have to do anything
special to talk about functors.


# Java

```java
// types ain't strong enough :(
interface Functor<F> {
    <A, B> F<B> fmap(Function<A, B> f, F<A> xs);
}

class ListFunctor<List> { ... }
```

Note:

Java's type system doesn't actually let us implement functor as an interface,
because can't have generic types like this. Note that F is the type variable in
the interface, and it's *unapplied*. Java just has no idea what to do with this
unfortunately. Maybe Java will get these kinds of generics in a future version.


# Haskell

```haskell
class Functor f where
    fmap :: (a -> b) -> f a -> f b

instance Functor Maybe where
    fmap _ Nothing =
        Nothing
    fmap f (Just a) =
        Just (f a)
```

Note:

This is the Haskell definition. We use a thing called a type class, which is
similar to an OO interface. Then we can *enroll* types to be instances of this
type class and provide the required functions, which lets us `fmap` over
different kinds of stuff.


# Containers

Note:

No, not like docker, sorry.
Containers are another common functor that we all deal with.
Or, rather, we *want* them to have the functor interface when we map over them!


### Categories involved:

## Functions! 

A mapping from

$$a \to b$$

to

$$F(a) \to F(b)$$

where $F$ is `List`, `Maybe`, `Set`, `Dictionary key`, etc...


```haskell
map length ["a", "bb", "ccc"]
-- => [1, 2, 3]
```

Note:

We want the function to not change the size of the container.
And we want to be able to compose things nicely.


```haskell
    map plusOne . map timesTwo . map length

                     ===

       map (plusOne . timesTwo . length)
```

Note:

The first line has the exact same type, and, since we're talking about functors, has the exact same result.
However, the top version will have to traverse the list three times -- once for each map!
The bottom version does a single traversal.
We know this is good because the functor laws guarantee it, so we can make the optimization safely.


# From Bash

# To Web Services?
<!-- .element: class="fragment" -->


# Constraining our Implementation

Note:

Let's implement map for lists now, keeping the functor laws in mind.


```haskell
data List a = Nil
            | Prepend a (List a)

listMap :: (     a ->      b)
        -> (List a -> List b)
listMap ...
```

Note:

This is our definition for the list type, and our function signature for the mapping function.
`listMap` takes a function from some generic type a to a generic type b, and returns a function from a list of as to a list of bs.


```haskell
data List a = Nil
            | Prepend a (List a)

listMap :: (a -> b)
        -> List a 
        -> List b
listMap f Nil = ...
```

Note:

Since Haskell automatically curries the functions, this is another way to represent the same thing.
It's a function that takes an `a -> b`, a List a, and returns a List b.
We can handle the empty case first. This one is easy.
We pattern match on the `Nil` constructor of the list.
What can we possibly return to match the types?
The only two constructors we can use are Nil and Prepend, and we don't have anything of type b to use with prepend.
So we'll have to use Nil.


```haskell
data List a = Nil
            | Prepend a (List a)

listMap :: (a -> b)
        -> List a 
        -> List b
listMap f Nil = Nil
listMap f (Prepend head tail) = ...
```

Note:

So what are our possibilities for this case? Well, let's try a dumb one:


```haskell
listMap f Nil = Nil
listMap f (Prepend head tail) = Nil
```

Note:

So this -- well, it fits the type! we've returned a list of type b.
But it is obviously the *wrong* thing to do. And we know this because it violates the functor laws!


# Identity in Haskell

```haskell
id :: a -> a
id x = x

listMap id [1, 2, 3] 
  ===
id [1, 2, 3]
```


```haskell
-- currently,
listMap id [1, 2, 3] 
  === 
Nil
  =/=
id [1, 2, 3]
  ===
[1, 2, 3]
```


# Back on track...

```haskell
listMap f Nil = 
    Nil
listMap f (Prepend head tail) = 
    Prepend (f head) ...
```

Note:

Ok, so we can't use Nil. We have to use the Prepend constructor.
We don't have any `b`s, lying around, but we do have an `a` and a way of transforming an `a` into a `b`.
How can we make a list of `b`s now?
Well, we could do the dumb thing and use the `Nil` constructor.
That fits the type, but it breaks the law again.
If we're not going to use Nil, then we need to use the list of as.
We can just call map recursively.


```haskell
listMap f Nil = 
    Nil
listMap f (Prepend head tail) =
    Prepend (f head) (listMap f tail)
```

Note:

And this one satisfies all our laws! 
We were able to find our way to a correct implementation of map just by taking
care to not break the laws and following the types. It's not often that we can
find such an easy way to know that we're right, and these laws give us that.


# Noticing Patterns

Note:

Here's another interesting functor:


```haskell
type Function a b = a -> b

instance Functor (Function r) where
    fmap :: (a -> b)
         -> Function r a
         -> Function r b
    fmap f g = ...
```

Note:

Functions! We can make the types look a little nicer like this. So what is the
implementation going to be? The first argument to `fmap` is a function from a
to b. We're mapping that function over to this other category, of functions
that take an `r`.


```haskell
type Function a b = a -> b

instance Functor (Function r) where
    fmap :: (a -> b)
         -> Function r a
         -> Function r b
    fmap f g = \r -> f (g r)
```

Note:

This looks familiar!
So functions are functors, and mapping is composition!

Isn't that funny?


more fun facts


# Functors...

## compose
<!-- .element: class="fragment" -->

## have an identity
<!-- .element: class="fragment" -->

Note:

A functor, like a function, is a mapping. And these mappings compose.


# category category

- objects: categories
- arrows: functors

Note:

Categories form a category!


Can we map functors?

# Yes we can!
<!-- .element: class="fragment" -->

Note:

Natural transformations


# Natural transformation

```haskell
type NaturalTransformation f g = 
    forall a. f a -> g a
```

Note:

A natural transformation is something with a type like this.


```haskell
listToMaybe :: NaturalTransformation [] Maybe
listToMaybe (x:_) = Just x
listToMaybe []   = Nothing
```


# Composes...

# Identity...
<!-- .element: class="fragment" -->

Note:

Well, we've seen this a bunch now...
The category category has categories as objects and functors as arrows.


# Functor Category

- Objects: Functors
- Morphisms: Natural Transformations
