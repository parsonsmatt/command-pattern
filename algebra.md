<!-- .slide: data-background="algebraic.gif" -->

Note:

Alright, now that we know about sets, we can start talking about stuff that *uses* that idea to build more complex ones!


# Algebra

### Three things:
<!-- .element: class="fragment" -->

- A set of objects.
<!-- .element: class="fragment" -->
- A set of operations closed over those objects.
<!-- .element: class="fragment" -->
- A set of laws or axioms governing the interaction of those operations.
<!-- .element: class="fragment" -->

Note:

Algebras consist of a set of objects, sometimes called the "carrier set" or "underlying set",
one or more operations closed over the objects,
and the laws governing these things.


## Informally,

# A game!
<!-- .element: class="fragment" -->

- A set of players/pieces/cards
<!-- .element: class="fragment" -->
- A set of moves/techniques/attacks
<!-- .element: class="fragment" -->
- The rules that govern how those all work together
<!-- .element: class="fragment" -->

Note:

Any game is an algebra. Sensible games have simple algebras.


<!-- .slide: data-background="dnd-rulebooks.png" -->

Note:

A complex game, like Dungeons and Dragons, has 1000s of pages written about the rules, operations, and objects in the game.


<!-- .slide: data-background="2048.gif" -->

Note:

And simple games, like 2048, tic tac toe, or agar.io, have very simple algebras.


# Denotational Design

- The art of designing software around algebras
- [Denotational Semantics (Wikipedia)](https://en.wikipedia.org/wiki/Denotational_semantics)
- [Denotational design with type class morphisms](http://conal.net/papers/type-class-morphisms/)

Note:

Designing an algebra for your problem and implementing the algebra is part of
a design philosophy called 'denotational semantics', which somewhat reminds me
of domain driven design.
