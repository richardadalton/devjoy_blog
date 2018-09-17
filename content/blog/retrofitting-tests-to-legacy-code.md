+++
date = "2011-06-13T12:00:00Z"
title = "Retrofitting Tests To Legacy Code"
tags = ["testing"]
categories = ["thoughts"]
banner = "img/banners/temple-stilts.jpg"
+++

One problem with TDD is that those who try it, often begin by writing a few trivial tests to get the lie of the land. Then instead of using TDD to write new code they do something much much harder. 

They do what I did, they start out by trying to write some unit tests for existing code. Either a past project, or more likely the project they are currently working on. Then they discover that these projects are almost impossible to test.

For some, that's all the evidence they need that TDD is fine in theory, but not practical for real work. But that reasoning is a little flawed.

They are new to writing automated tests, which is a tricky enough skill to master. They are also new to writing testable code which is actually a much bigger challenge. The chances that their legacy code is testable is pretty much nill.

Their first use of TDD poses an obstacle that would be a challenge for those with more experience of TDD.

Think a video game with the ultimate Boss level as the very first thing you do. That right there is a game that you’ll throw in the bin after about an hour, if you last that long.

If you're new to TDD, don't judge it by how easy it is to retrofit to existing code. And, if you're retrofitting tests to existing code, don't expect it to be easy.

Here's a [Stack Overflow question](https://stackoverflow.com/questions/606811/tdd-and-di-dependency-injections-becoming-cumbersome/616589#616589) on how to write tests hard to test code. The answers cover practical solutions and pragmatic advice.

Robert Martin contributes an interesting Step By Step on how to add tests to existing code (in an kludgy way). He then explains how to refactor both the original code and the kludgy test handling.

#### Recommended Reading
[Working Effectively With Legacy Code - Michael Feathers](http://www.bookdepository.com/book/9780131177055/?a_aid=richardadalton)

