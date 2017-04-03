# I Command You

## To Be Free!

http://www.parsonsmatt.org/command-pattern

Note:

Hi! I'm Matt Parsons, and I'd like to talk to you about some cool software
practices I've been using recently to make better software.

If you want to follow along with the slides, they're available at the URL above.


# Better?

* Performance?
<!-- .element: class="fragment" -->
* Correctness?
<!-- .element: class="fragment" -->
* Ease of understanding?
<!-- .element: class="fragment" -->
* Ease of reuse?
<!-- .element: class="fragment" -->
* Easy to modify?
<!-- .element: class="fragment" -->
* Easy to test?
<!-- .element: class="fragment" -->

Note:

Of course, better is subjective. Faster code is better, but performance isn't
free -- you have to spend time implementing it.  Code that gives the right
answer is important, but sometimes "close enough" is good enough.

Ease of understanding is important and subjective. I personally have no issue 
understanding Haskell code, and may trip up with complex Java hierarchies or
JavaScript scoping rules that are perfectly obvious and idiomatic for a
familiar programmer.

Ease of reuse is a little easier to understand. How hard is it to repurpose
this code for other related tasks? This is difficult to know without actually
reusing the code, which may be too late to understand how good the code
actually is.

Modifiability is important. If business rules need to change, then the code
needs to change. How difficult this is determines how quickly the business react
to changing requirements.

Testability -- this is a really good metric for good code!


# Test Driven Design

## aka
<!-- .element: class="fragment" -->

### If it sucks to test,
<!-- .element: class="fragment" -->

### it sucks
<!-- .element: class="fragment" -->


# Test Driven Design

* Informs API/library design
* Immediate feedback on code reusability
* Free regression/unit testing

Note:

Writing tests first is a great proxy for writing good code. Having "tests" to
cover correctness and find bugs is just about a nice side effect. You get
immediate feedback on your library design. If it sucks to write tests for your
code, it probably sucks to use your code.


# TDD is hard!

Note:

Writing test-first is *really* difficult. This is partially because writing
"better" code is really difficult.


# This isn't a TDD talk

Note:

I don't care about TDD. Testability is a nice *proxy* for "good" code.


<!-- .slide: data-background="finger-moon.jpg" -->
Note:

Code that's easy to test tends to be easy to modify, update, reuse, and verify.
It is like a finger pointing at the moon of good software: don't look at the
tests, look at the good software! You don't have to write tests to write good
code, and just because you wrote tests doesn't mean your code is nice.
