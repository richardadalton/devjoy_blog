+++
date = "2012-10-28T17:43:00Z"
title = "Are Unit Tests the new Comments?"
tags = ["testing"]
categories = ["thoughts"]
banner = "img/banners/banner-1.jpg"
+++

It’s verging on heresy to even talk of Unit Tests and Comments as being in any way related. They serve different purposes, work in different ways, and have nothing in common.

Except

### Comments
* Document Interfaces, API’s etc.
* Can drive development by writing pseudo code comments first.
* Mark outstanding work using TODO comments.
* Explain particularly complicated pieces of code.
* Context to help future developers avoid breaking the code.
* Document expectations, side-effects etc.
* Can get out of sync with the code.

### Unit Tests
* Document Interfaces, API’s etc.
* Can drive development by writing tests first.
* Mark outstanding work using failing tests.
* Help tease out particularly complicated algorithms.
* Tell future developers when they've broken the code.
* Test expectations, side-effects etc.
* Likely to fail if out of sync with the code.

OK, I’m using a little bit of poetic license here to highlight the similarities. You can find a handful of similarities between almost anything if you look hard enough. Nonetheless, there are similarities. Automated tests solve many problems that comments have used to tackle.

### From Mandatory to Code Smell
Once upon a time comments were pretty much mandatory. Coding standards included rules and guidelines on how to write them. Every file would contain comments as a header.

Today, there are mutterings of discontent about comments. There are some who go as far to describe them as a code smell. 

Better development tools have reduced the reliance on comments.
The drive for self-documenting, clean code has further undermined their selling point.

The notion of aiming for “comment coverage” is absurd. Most of us try to write code that doesn't need further explanation. The idea that we should then add comments to that code "just because" is absurd.

So, we need comments, but less than we used to. There are still uses for comments. 

If you see comment explaining *why* code is like it is, that might be ok. 

If you see comments explaining *what* code does, or *how* it does it, that might be a code smell.

### Unit Tests as a Code Smell
Can unit tests ever be a code smell? It almost seems like a ridiculous question. The big problem most of us seem to have with unit testing is not doing it enough. Is it beyond the pale that we should be doing less?

There was a time when the only problem with comments was not having enough. The very idea of too many comments was laughable. But here we are.

Can the same happen with unit tests?

We’ve already moved past the idea of thresholds for test coverage (at least some have).

We have integration and acceptance tests that can map to deliverable features. They also provide a means of catching regression bugs.

Why is testing individual code ‘units’ considered a fundamental good? Aren’t such units by their very nature implementation details? And if they are, why are we testing implementation details?

Aha! but TDD. Unit tests have a function in driving out the development of complicated functionality? But implicit in this is the idea that tests have their place. Much like comments. Finding tests somewhere that isn't "their place" may be a code smell.

### Keeping in sync
Whether tests come to be regarded as a code smell may hinge on how we keep them in sync with the code they test.

The ease with which comments and code could diverge from each other is a serious problem.

Automated tests can tell you when they are out of sync with the code.

However, while it might be easier to discover divergence, the holy grail is avoiding it in the first place. It's a fact of life that code changes, implementation details change. If you test implementation details, your tests will have to change.

The lower level the test, the more likely it is that changes to the code will mean changes to the test.

### Less is More
Tests are code. Generally speaking, if you can deliver the same value with less code, then that’s a good way to go. If we can get most of the benefits of unit tests, while writing less tests at higher levels, isn't that preferable?

The question is can we? Are there inherent benefits of unit tests that can’t be either replicated by higher level tests? or mitigated by technology or techniques?

I have no idea what the future holds for software development, or for unit tests. History tells us that today’s best practice can become tomorrow’s code smell.