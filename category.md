# Category

Like a more general set!


<!-- .slide: data-background-size="contain" data-background="category-1.jpg" -->

Note:

Categories are like sets, in that they have objects. Unlike sets, we don't
allow random membership in the category -- and, in fact, we usually can't talk
much about the objects at all!


<!-- .slide: data-background-size="contain" data-background="category-2.jpg" -->

Note:

Where sets have functions, categories have arrows. The arrows are also often
referred to as morphisms. While functions in sets can map to other sets, Arrows
in a category always map objects in the category to objects in the same
category.


<!-- .slide: data-background-size="contain" data-background="category-3.jpg" -->


# law and order

To establish a category:


<!-- .slide: data-background-size="contain" data-background="category-2.jpg" -->

Note:

The first law is identity. Every object in the category must have an arrow that
points back to itself. Remember, we get to pick whatever we want the arrows to
be.


<!-- .slide: data-background-size="contain" data-background="category-4.jpg" -->

Note:

If we've got an arrow from A to B, and an arrow from B to C, then we get an
implicit *composed* arrow from A to C. 


<!-- .slide: data-background-size="contain" data-background="category-5.jpg" -->

Note:

It also must be the case that the diagram drawn *makes sense*. Any path we take
through the diagram should have the same end result, with respect to where we
end up.


# Essence of:

# Composition

Note:

This formal definition of composition gives us exactly what we need to be able
to talk rigorously about composition. What are some categories?

Unfortunately, Java is unable to express these ideas in the type system as-is.
It lacks the ability to talk about generic types that are parameterized on
another generic type.


# Functions

Identity:

```haskell
type Function a b = a -> b

identity :: Function a a
identity a = a

compose  :: Function b c 
         -> Function a b 
         -> Function a c
compose f g = \x -> f (g x)

f . g = \x -> f (g x)
```

Note:

To show that something is a category, we have to provide the thing satisfying
identity and the thing satisfying composition.


```haskell
length      :: String -> Int
addOne      :: Int -> Int
replicate   :: a -> Int -> [a]

wat :: String -> [Int]
wat = replicate 3 . addOne . length
```


# Stream Processing

Stream processing libraries generally have:

1. Producers
2. Processors
3. Consumers

Note:

Producers generate data to be modified and consumed later.
Producers might be a list, database query, HTTP request, etc.

Processors modify each item one at a time, filter the stream, accumulate items and do some aggregation, etc.

Consumers receive each event and do something with it, like save to disk/database, print an OK, etc...


# Processors are a category


# Identity

```python
def stream_identity(stream):
    for x in stream:
        yield x
```

Note:

Identity is easy. We take a stream, and then for each item in the stream, we yield it up.
So the function `stream_identity` takes a generator and returns a generator.


# Composition

```python
def compose(proc_1, proc_2):

    def go(stream):
        for x in proc_1(proc_2(stream)):
            yield x

    return go
```

Note:

Composition is a little trickier. We have to keep the interface the same.
So we have to take two processors as arguments, and then the return type has to also be a processor.
What is a processor: a function that takes a generator and returns a new generator!


# All monoids are categories

Note:

If you've got a monoid, you've got a very simple category.


# Bash Pipes


# Composition:

# |
<!-- .element: class="fragment" -->

Note:

Composition is easy. You all know and love this one.
It's the bash pipe!
Some of you know what the identity is, too, I bet.


# Identity:


<!-- .slide: data-background="cat-pipe-1.jpg" -->


<!-- .slide: data-background="cat-pipe-2.jpg" -->


<!-- .slide: data-background="cat-pipe-3.jpg" -->


<!-- .slide: data-background="cat-pipe-4.jpg" -->


<!-- .slide: data-background="cat-pipe-5.jpg" -->


<!-- .slide: data-background="cat-pipe-6.jpg" -->


`cat`


`cat **/*.html | wc -l`

`wc -l **/*.html`
<!-- .element: class="fragment" -->

`cat **/*.html | cat | cat | cat | wc -l`
<!-- .element: class="fragment" -->


# Categorical web service?


## why not?

Note:

So we need to choose our objects. Responses/requests seem sensible.
We also need our morphisms. Endpoints seem rather natural for that.


# Identity:

```haskell
# Request:

GET /echo?param=123
User-Agent: ...
Host: ...
...
```


```haskell
# Response:

HTTP/1.1 200 OK
Date: ...
Server: ...
Last-Modified: ...
Content-Length: 3
Content-Type: text/text
Connection: Closed
123
```

Note:

So the identity endpoint is just like an echo. It returns whatever you send it.


# Composition:

blah http requests are boring


<!-- .slide: data-background-size="contain" data-background="round-trip.png" -->

Note:

The composition of endpoints sends a request to an endpoint, gets a response, sends that response to another endpoint, and then gets the final response back.


that's an awful lot of round tripping

Note:

So that's a lot of round trips.
We *know* that the server implements these endpoints as code somehow.
And for it to be able to do that with our request and responses implies that the backend is a category too, of some sort.
It seems like we should be able to make a composed request, right?
