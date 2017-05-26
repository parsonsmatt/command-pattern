# I Command You

## To Be Free!

Matt Parsons


![](image_MattParsons.jpg) <!-- .element: id="plain" -->
![](seller-labs-dark.png) <!-- .element: id="plain" -->

Note:

Hi! I'm Matt Parsons.
So this is me on the internet, and now I'm me in real life.
I write Haskell, PHP, and JavaScript for Seller Labs.
We make web applications to help Amazon merchants make more money.

I'm going to share a technique we've been using in our PHP codebase to help us write better code.


# Better code?

* Performance?
<!-- .element: class="fragment" -->
* Correctness?
<!-- .element: class="fragment" -->
* Maintainability?
<!-- .element: class="fragment" -->
* Testability?
<!-- .element: class="fragment" -->

Note:

Of course, better is subjective. Faster code is better, but performance isn't
free -- you have to spend time implementing it.  Code that gives the right
answer is important, but sometimes "close enough" is good enough.

Maintainability is another good point. How easy is the software to maintain,
refactor, and modify? If busienss rules need to change, then how difficult will
it be to make these logical changes? Unfortunately, this is difficult to
understand without actually attempting to do the change, so it's often too
late.

Testability -- this is a really good metric for good code!
Testing code is often the first time that you go to actually use the code you wrote.
It's the first time you have to setup the surrounding code and infrastructure, and you get immediate feedback on how good your code is to reuse.


# Test Driven Design

## aka
<!-- .element: class="fragment" -->

### If it sucks to test,
<!-- .element: class="fragment" -->

### it sucks
<!-- .element: class="fragment" -->

Note:

Test driven development is essentially the idea that:
If it sucks to write tests for your code, it probably sucks to use your code.


# Test Driven Design

* Informs API/library design
<!-- .element: class="fragment" -->
* Immediate feedback on code reusability
<!-- .element: class="fragment" -->
* Free regression/unit testing
<!-- .element: class="fragment" -->

Note:

Writing tests first is a great proxy for writing good code.
You get immediate feedback on your library design.
Having tests to cover correctness and find bugs is just about a nice side effect. 


# TDD is hard!

Note:

Writing test-first is *really* difficult.
This is partially because writing code that is nice to test (and, by proxy, reuse/modify/understand/etc) is really difficult.


# This isn't a TDD talk

Note:

I don't care about TDD. Testability is a nice *proxy* for "good" code.


<!-- .slide: data-background="finger-moon.jpg" -->
Note:

Code that's easy to test tends to be easy to modify, update, reuse, and verify.
It is like a finger pointing at the moon of good software: don't look at the
tests, look at the good software! You don't have to write tests to write good
code, and just because you wrote tests doesn't mean your code is nice.
