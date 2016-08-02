### Set:

$$0, 1, 2, 3, 4, 5, 6, 7, 9, 10, 11, 12...$$

### operation: $+$

$$3 + (4 + 6) = 13$$

$$(3 + 4) + 6 = 13$$

$$0 + 6 = 6$$

$$4 + 0 = 4$$

Note:

Let's get our feet wet with a specific algebra now.
Let's try to find a pattern...


### Set:

$$0, 1, 2, 3, 4, 5, 6, 7, 9, 10, 11, 12...$$

### operation: $\times$

$$2 \times (3 \times 6) = 36$$

$$(2 \times 3) \times 6 = 36$$

$$1 \times 3 = 3$$

$$9 \times 1 = 9$$


### Set:

`"", "a", "b", "ab", "c", "abc", ...`

### operation: $.$

`"hello" . (" " . "world") = "hello world"`

`("hello" . " ") . "world " = "hello world"`

`"" . "hey" = "hey"`

`"foo" . "" = "foo"`


### Set:

```ruby
{}, { a: 1 }, { a: 2 }, { b: 1 }, { c: 3 } ...
```

### operation: `merge`

```ruby
{ a: 1 }.merge( { b: 2 }.merge({ c: 3 }) ) = 
    { a: 1, b: 2, c: 3 }
{ a: 1 }.merge({ b: 2 }).merge({ c: 3 }) =
    { a: 1, b: 2, c: 3 }
```

```ruby
{ a: 1 }.merge {} = { a: 1 }
{}.merge { a: 1 } = { a: 1 }
```


# Monoid

## Algebra, so it has the three things:
<!-- .element: class="fragment" -->

- Set of objects:
<!-- .element: class="fragment" -->
- Set of operations:
<!-- .element: class="fragment" -->
- Set of laws:
<!-- .element: class="fragment" -->


# Objects:

You choose a set!
<!-- .element: class="fragment" -->

It has to have an "identity" object.
<!-- .element: class="fragment" -->


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
    M combine(M a, M b);
    M identity();
}
```

```java
class PlusMonoid implements Monoid<Integer> {
    Integer identity() {
        return 0; 
    }

    Integer combine(Integer a, Integer b) {
        return a + b; 
    }
}
```
<!-- .element: class="fragment" -->


laundry time

```java
static <M> M fold(
    Monoid<M> monoid, 
    Stream<M> objects
    ) {
    return objects.reduce(
        monoid.identity(), 
        monoid::combine
    );
}
```

Note:

This is fold. If we have some stream of M values, and we have our monoid instance for those Ms handy, then we can combine them arbitrarily.
The monoid property gives us a nice benefit in terms of performance.


# sum

```java
static Integer sum(Stream<Integer> numbers) {
    return fold(new PlusMonoid(), numbers);
}
```
<!-- .element: class="fragment" -->

Note:

Implementing sum via the monoid is pretty easy.

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


# $$a \otimes b \otimes c \otimes b \otimes d \otimes e \otimes$$

Note:

So suppose we have some string of operations like this that we need to perform.
How do we evaluate it? It depends on how the operator *associates*. If it associates to the right, then we have to start at the rightmost expression and work our way up.


# $$a \otimes (b \otimes (c \otimes (b \otimes (d \otimes (e \otimes$$

Note:

This is what the expression looks like. It's just this big deep chain of operators.


# $$1 - 2 - 3 - 4 - 5 - 6 - 7$$

Note:

Subtraction, for example, is not associative, so the parentheses matter, and we have to perform the operations linearly.


## $1 - (3 - 5) = 3$

## $(1 - 3) - 5 = -7$


<!-- .slide: data-background-size="contain" data-background="minus-tree.jpg" -->


# $$1 + 2 + 3 + 4 + 5 + 6 + 7$$

Note:

Now, with a monoid, we can rearrange the parentheses however we want.
This means that we get to choose the order of evaluation for how we combine the elements.


<!-- .slide: data-background-size="contain" data-background="binary-eval-tree-2.jpg" -->

Note:

The stuff on the left of this tree doesn't depend on the stuff on the right.
It can be easily parallelized.
So if we know we have a monoid, we can use that to significant performance benefit.


<!-- .slide: data-background-size="contain" data-background="binary-eval-tree-3.jpg" -->


<!-- .slide: data-background-size="contain" data-background="binary-eval-tree-4.jpg" -->


<!-- .slide: data-background-size="contain" data-background="binary-eval-tree-5.jpg" -->
