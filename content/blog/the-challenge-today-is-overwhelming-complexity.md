+++
date = "2014-08-18T17:43:00Z"
title = "The Challenge Today Is Overwhelming Complexity"
tags = ["complexity"]
categories = ["thoughts"]
banner = "img/banners/complexity.png"
aliases = [
    "/2014/08/the-challenge-today-is-overwhelming-complexity/",
]
+++

Here’s a really great [post](http://blog.moertel.com/posts/2013-12-14-great-old-timey-game-programming-hack.html) by [Tom Moertel](https://plus.google.com/+TomMoertel) on squeezing every last ounce of performance out of machines back in the day.

It was a time when unrolling a loop to save the few clock cycles or seeing a unique way to use the registers of a chip could take a game from clunky to classic.

Stories of making machines do the impossible are the stuff of legend. The closest we mere mortals came was rejigging our config.sys and autoexec.bat files to free up a little memory. If you’re under 30 even that may sound alien.

Tom’s post is a great read, but there is a real gem of wisdom towards the end, under the heading “To the old days!”.

“The challenge wasn’t overwhelming complexity, as it is today. The challenge was cramming your ideas into machines so slow, so limited that most ideas didn’t fit.”

Turn that quote around. The challenge today isn’t cramming your ideas into machines so slow so limited that most ideas don’t fit. The challenge today is overwhelming complexity.

<blockquote>The challenge today isn’t cramming your ideas into machines so slow so limited that most ideas don’t fit. The challenge today is overwhelming complexity.</blockquote>

I would guess that at this point less than 5% of developers ever need to sacrifice maintainability for performance. I’m probably being generous at that. If you are taking on complexity in order to speed up the execution of a method, or reduce it’s memory footprint, you’re more likely doing harm for no good reason.

But, this post isn’t about code. It’s all too easy to focus on “Clean Code”, have code reviews, refactor relentlessly etc. But, if you deploy that code into an overly complex environment, or using an unnecessarily complex infrastructure, or manage the whole thing using unnecessarily complex ceremony then your code cleaning efforts may be wasted.

It’s interesting how many books are available on how to write “good code”, or how to refactor “bad code”, but there are relatively few if any books about refactoring “bad environments”.

Refactoring should extend to the entire development process. If you can refactor away an unneeded Class, but can’t get rid of an unneeded tool, you’re on the road to ruin. No amount of intention revealing Interfaces or SOLID code will save you.

The phrase “Technical Debt” has been coined to mean the deliberate taking on of subpar “code”, with a view to fixing that problem later. I don’t believe it’s a satisfactory definition.

All too often “Technical Debt” means “We have this one line fix, we’ll come back later and implement it correctly in all of the Controllers, Views and Models that should be affected. I wonder about the logic of treating a change in multiple places as preferable to making a change in one place.

I also wonder whether that one liner change is where time will be lost in maintenance. Even if there is a more “correct” way of fixing an issue, is the code really where your team is losing productive hours?

Or, is it the time needed to set up a development environment? Or figure out how to find a test environment that’s “close enough” to production to be useful. Perhaps you have so many tools that nobody really understands how everything fits together any more.

If there is a “Knack” to getting your code base to build, then clean code is the least of your problems.

When the complexity of the solution exceeds the complexity of the problem, THAT’s technical debt. By “solution” I mean the WHOLE package. Code, Tools, Frameworks, Process, Infrastructure, Documentation, Planning, Review.

Very few of us need to find “clever” solutions any more. The capabilities of machines far exceed almost anything we could possibly want to do. The challenge now is complexity and we should be devoting a lot more of our attention to it.