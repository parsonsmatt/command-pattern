# Let's talk about sets,

# baby
<!-- .element: class="fragment" -->

## Let's talk about 
<!-- .element: class="fragment" -->

###you and me
<!-- .element: class="fragment" -->


# Set as data structure

Note:

Sets are like dictionaries/hashes/maps, but without values.

They have very fast insertion, membership checking,
union, intersection, difference.

Examples of use:


# Set Theory

Note:

So what is a set?

A set is a collection of objects.
Sets don't have a notion of ordering.
Sets also don't have duplicate objects


*drawing: set of random things*

Note:

Sets can be totally random objects, like the set consisting of Brazil, my
neighbor Dan, three unique different cats, and a crayon.


*drawing: set of numbers*

Note:

It's usually more useful to talk about sets that have some regular properties,
that we can derive some rule.


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


*drawing: set*


*drawing: set + endomorphism*


*drawing: two sets*


*drawing: sets + function mapping from a to b*

Note:

Like, consider `length : String -> Int`.
