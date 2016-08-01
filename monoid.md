*picture of int, 0, +*

Note:

Let's get our feet wet with a specific algebra now.
Let's try to find a pattern...


*picture of int, 1, times*


*picture of strings, "", ++ *


*picture of dictionary, merge*


*picture of boolean, true, &&*


# Monoid

## Algebra, so it has the three things:

- Set of objects:
- Set of operations:
- Set of laws:


# Objects:

You choose a set!
It has to have an "identity" object.


# Operations:

Choose some binary operator on the objects you supplied.
We'll generically call it: $\oplus$


# Laws:

## Associativity:

$$x \oplus (y \oplus z) = (x \oplus y) \oplus z$$

## Identity:

$$x \oplus identity = x$$

$$identity \oplus x = x$$


<!-- .slide: data-background="y-tho.jpg" -->

Note:

This is a nice pattern, but so what? Why do we care about it? The first is that
it can provide a very nice and familiar API. Just like you can say to
a coworker "This is a factory", you can say "this is a monoid", and they'll
know you can combine it without worrying about order.


# Java

```java
interface Monoid<M> {
    public M combine(M a, M b);
    public M identity();
}
```

```java
class PlusMonoid implements Monoid<Integer> {
    public Integer identity() {
        return 0; 
    }

    public Integer combine(Integer a, Integer b) {
        return a + b; 
    }
}
```
<!-- .element: class="fragment" -->


laundry time

```java
static <M> fold(Monoid<M> monoid, Stream<M> objects) {
    return objects.reduce(
        monoid.identity(), 
        monoid::combine
    );
}
```

```java
// Can be more generic!
static <M, F> fold(Monoid<M> m, Reducible<F<M>> objects) { ... }
// ... But Java doesn't let us talk about generics of generics...
```
<!-- .element: class="fragment" -->


# sum

```java
static Integer sum(Stream<Integer> numbers) {
    return fold(new PlusMonoid(), numbers);
}
```
<!-- .element: class="fragment" -->


# product

```java
class ProductMonoid implements Monoid<Integer> {
    public Integer identity() {
        return 1; 
    }

    public Integer combine(Integer a, Integer b) {
        return a * b; 
    }
}

static Integer product(Stream<Integer> numbers) {
    return fold(new ProductMonoid(), numbers);
}
```
<!-- .element: class="fragment" -->


# Essence Of:

# MapReduce


```java
static <M, A> M mapReduce(
        Monoid<M> monoid,
        Stream<A> values, 
        Function<A, M> mapper) {
    return fold(monoid, values.map(mapper::apply);
}
```

```haskell
mapReduce :: Monoid m => (a -> m) -> [a] -> m
mapReduce func values = foldr (map func values)
```
<!-- .element: class="fragment" -->


*picture: linear operation*

Note:

So suppose we have some string of operations like this that we need to perform.
If we don't know any better, then we have to respect however the parnetheses are.


*picture: 1 - 2 - 3 - 4*

(3 - 4) - 5 != 3 - (4 - 5)

Note:

Subtraction, for example, is not associative, so the parentheses matter, and we have to perform the operations linearly.


*picture: 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10*

Note:

Now, with a monoid, we can rearrange the parentheses however we want.
This means that we get to choose the order of evaluation for how we combine the elements.


*picture: binary tree operation*

Note:

The stuff on the left of this tree doesn't depend on the stuff on the right.
It can be easily parallelized.
So if we know we have a monoid, we can use that to significant performance benefit.
