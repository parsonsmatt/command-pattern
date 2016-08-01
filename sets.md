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


Example

*picture: stream of requests*

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


*drawing: set of random things*

Note:

Sets can be totally random objects, like the set consisting of Brazil, my
neighbor, three different cats, and a crayon.


*drawing: set of numbers*

Note:

It's usually more useful to talk about sets that have some regular properties,
that we can derive from some rule.


```haskell
-- Haskell

evens = [ n | n <- [1..], even n ]
```

```python
# Python

evens = { n for n in itertools.count() if n % 2 == 0 }
```

Note:

Some programming languages have "list comprehension" syntax, which lets you
declaratively talk about collections as a source and a bunch of rules
describing the elements contained therein.

Python lets you have list, set, *and* dictionary comprehension syntax, which is
pretty great.

It's actually pretty convenient to think about *types* like they were sets.


*drawing: set of all numbers*

Note:

Like the set of all numbers, or the set of all characters,


*drawing: set of all characters*


*drawing: set of all strings*

Note:

Or the set of all strings, which is really a special case of the set of all
lists


*drawing: set of all lists*


# Functions

Note:

Before too long, we want to talk about transforming elements.
A function is something that *maps* an element from a set onto another element,
potentially in the same set, but maybe in a different one.


*drawing: set braces*

Note:

So we're going to get a little abstract now. This is our set notation.


*drawing: set 'A' with objects*

Note:

And now we've got these objects that live in the set.
That's all we know about them.
We're going to call the set 'A'.


*drawing: set + endomorphism*

Note:

So now we have this arrow -- we'll call it 'f' -- that takes an object in A and maps it to another object in A.
For some arrow to be a function, it has to always map the same start point to the same end point.


*drawing: set + two endomorphisms*

Note:

Otherwise it's just obviously not the same arrow, and therefore not the same function.


## Scary Word Time

# Endomorphism
<!-- .element: class="fragment" -->

Note:

So, this idea of an arrow or function is also referred to as a "morphism."
And if we have a function that maps objects from a set back to the same set, then we call that an "endomorphism."
Aside: If your intuition on 'morphism' is "isomorphic web apps," then I'm sorry but you've been led astray.
An isomorphism is when you have a pair of functions which can "round trip"


*drawing: two sets, A and B*

Note:

So this is a pair of sets, we'll call A and B.


*drawing: sets + function mapping from a to b*

Note:

Like, consider `length : String -> Int`.
We have the set of strings, and the set of ints.
`length` maps from one set to the other.
