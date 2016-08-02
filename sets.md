# Let's talk about sets,

# baby
<!-- .element: class="fragment" -->

## Let's talk about 
<!-- .element: class="fragment" -->

###you and me
<!-- .element: class="fragment" -->


By analogy...

```ruby
# Ruby:
hash = { foo: "bar", baz: 123 }
hash[:foo] # => "bar"
```
<!-- .element: class="fragment" -->

```javascript
// JavaScript:
let obj = { foo: "bar", baz: 123 };
obj.foo // => "bar"
```
<!-- .element: class="fragment" -->

```java
// Java:
Map<String, Int> map = new HashMap();
map.insert("foo", 3);
map.get("foo"); // => 3
```
<!-- .element: class="fragment" -->

Note:

We can develop an intuition for sets by analogy with another data type that you may be more familiar with.
Ruby hashes, JavaScript objects, and Java maps are all examples of a dictionary/map type of data structure.
You insert keys and values, and keys are unique.


Sets:

```ruby
# Ruby:
set = Set.new [1, 2, "foo"]
```

```javascript
// JavaScript:
let set = new Set([1, 2, "foo"]);
```
<!-- .element: class="fragment" -->

```java
// Java:
Set<String> set = new HashSet();
set.add("Foo");
set.add("bar");
```
<!-- .element: class="fragment" -->

```python
# Python:
a_set = { 1, 2, 2, 3 } # actually {1, 2, 3}
```
<!-- .element: class="fragment" -->

Note:

Sets are like dictionaries/hashes/maps, but without values.
They have very fast insertion, membership checking, union, intersection, difference.
They are unordered.


<!-- .slide: data-background="request-stream.png" -->

Note:

Sets are useful when you need to keep track of uniqueness.
Suppose you've got a stream of requests, and you want to keep track of how many requests you service as well as the count of unique requests.
For the count, you just have a counter: +1
For the unique requests, you insert the request into the set, and query the size of the set.


<!-- .slide: data-background="mathematical.gif" -->


# Set Theory

$$\\{\\}$$

Note:

So it turns out sets are *really* useful as an abstract notion!
You can build basically all of math on top of this idea of an unordered container of unique objects.
It's usually more useful to talk about sets that have some regular properties,
that we can derive from some rule.


$$\\{ n | n \in \mathbb{N}, even?(n) \\}$$

```haskell
-- Haskell

evens = [ n | n <- [1..], even n ]
```

```python
# Python

even = { n for n in range(0) if n % 2 == 0 }
```

Note:

Some programming languages have "list comprehension" syntax, which lets you
declaratively talk about collections as a source and a bunch of rules
describing the elements contained therein.

Python lets you have list, set, *and* dictionary comprehension syntax, which is
pretty great.

It's actually pretty convenient to think about *types* like they were sets.


# Set of all numbers:

## $\mathbb{N} = \\{ 0, 1, 2, \dots n, n + 1 \dots \\}$

Note:

Like the set of all numbers, or the set of all characters,


# Set of all characters:

## $$\mathbb{C} = \\{a, b, c, d, e, f, g, h, i, j, k, l, m, n\\}$$

Note:

When talking about sets like these, we're kind of talking about types, like in
the type signatures for our functions.


# Functions

Note:

Before too long, we want to talk about transforming elements.
A function is something that *maps* an element from a set onto another element,
potentially in the same set, but maybe in a different one.


<!-- .slide: data-background="set-a-endo.jpg" -->

Note:

And now we've got these objects that live in the set.
That's all we know about them.
We're going to call the set 'A'.

So now we have these arrows -- we'll call them 'f' and 'g' -- that take an object in A and map it to another object in A.
For some arrow to be a function, it has to always map the same start point to the same end point.


<!-- .slide: data-background="set-a.jpg" -->

Note:

Otherwise it's just obviously not the same arrow, and therefore not the same function.


## Scary Word Time

# Endomorphism
<!-- .element: class="fragment" -->

Note:

So, this idea of an arrow or function is also referred to as a "morphism."
And if we have a function that maps objects from a set back to the same set, then we call that an "endomorphism."


<!-- .slide: data-background="two-sets-a-b.jpg" -->

Note:

So this is a pair of sets, we'll call A and B.


# Morphism


<!-- .slide: data-background="two-sets-arrow.jpg" -->

Note:

Like, consider `length : String -> Int`.
We have the set of strings, and the set of ints.
`length` maps from one set to the other.
