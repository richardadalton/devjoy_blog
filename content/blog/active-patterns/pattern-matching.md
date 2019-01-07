+++
date = "2014-08-07T00:00:00Z"
title = "Pattern Matching"
tags = ["functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png"
aliases = [
    "/2014/08/pattern-matching/",
]
+++

This is the first in a series of posts explaining Active Patterns, a very cool feature of F#. This post will lay the groundwork by covering pattern matching, and introducing the concept of active patterns. Subsequent posts will cover the various types of active pattern in detail.

### Destructuring Assignment
Thanks to [Miles McGuire](https://twitter.com/richardadalton/status/497350991331618816) for setting me straight on the name.

F# is full of little nice ideas that you appreciate when you come from a C# background, and destructuring assignment is one of them. Take a look at the following.

{{< highlight fsharp "style=tango" >}}
let date = (21, 8, 2014)

let d, m, y = date
val y : int = 2014
val m : int = 8
val d : int = 21

let _, month, year = date
val year : int = 2014
val month : int = 8
{{< / highlight >}}

That’s a tuple representing a date. To extract values from the tuple and assign them to the three variables d, m and y we can do a single assignment. The tuple gets deconstructed (think constructor in reverse).

In the second assignment we use the _ to indicate that we don’t care about the first value in the tuple. It will create two variables month and year.

That alone is pretty nice, but it gets better.

### Pattern Matching
Destructuring assignment is really a form of Pattern Matching, which is common in Functional Programming languages. In FSharp the match expression allows us to match against quite elaborate data structures.

First let’s take a look at a match expression that does little more than the assignment above.

{{< highlight fsharp "style=tango" >}}
let patternMatch date =
    match date with
    | d, m, y -> sprintf "Day [%d], Month [%d], Year [%d]" d m y   
{{< / highlight >}}

With Pattern matching we can deconstruct the date just as we did in the assignment. The match expression allows us to create multiple match clauses, with the value of the overall expression being decided by the first clause that matches.

{{< highlight fsharp "style=tango" >}}
let partialPatternMatch date =
    match date with
    | _, _, y when y < 1960 -> "Nothing before the 60's matters"
    | _, m, 1960 -> sprintf "Ah the month was %d, the 60's were just beginning" m
    | _, 12, y -> sprintf "It was Christmas, %d" y
    | d, m, y -> sprintf "Day [%d], Month [%d], Year [%d]" d m y
{{< / highlight >}}

Clauses can match based on a ‘when’ guard as in the first example above. We can also insert literal values into the pattern as with the year 1960 in the second case and the month 12 in the third. In both of those examples we match with any day because we use the underscore to indicate that we don’t care what the day is. Also in those examples we match the month and year respectively to variables (m and y) which can be used in the results.

In the final clause we are guaranteed to match any date that fails to match on any of the first three clauses. There are no literal values or when guards that could cause the last case to fail, we just get the day month and year, whatever they may be and can use them in the result.

### Match Against Abstractions
As exciting as all that is, you’ll notice that like the assignment earlier, the pattern matching in a match expression really only matches against and/or extracts data that’s already there. We’re matching here on the raw numbers that are the underlying representation of the date. It would be nice to match on things like

* May
* First Quarter
* Summer
* Month End
* Saturday
* Holiday

These are all abstractions that should be independent of how the date is implemented. Active Patterns allow us to pattern match against these kinds of abstraction, rather than the underlying data.

### Active Patterns
What we need is pattern matching's ability to extract and match against data, but with the added ability to transform the data we’re working with.

We already have a really good general purpose way of transforming data, namely functions, so it shouldn’t come as any surprise that Active Patterns are special kind of function with some extra powers that make them useful in pattern matching.

Active Patterns come in a few varieties so, the next couple of posts will describe the various options. Don’t worry too much about the notation below. All will become clear as you read through the examples.

* Single Total: (|A|)
* Single Total with Params: (|A|) x
* Single Partial: (|A|_|)
* Single Partial with Params: (|A|_|) x
* Multiple Total: (|A|B|)

OK, I haven’t explained what Total and Partial means yet. All in good time, but if you’re a little OCD like me you’ll notice that list doesn’t look quite complete, there is no “Multiple Partial”. The reason for that is very simple, the language designers haven’t added them yet, and may not add them at all.

> “For completeness our specification includes structured names with multiple cases, e.g, (|ParseInt|ParseFloat|_|). However we have yet to detect any practical benefit in doing so.”[1]

We’re getting ahead of ourselves. The [next post]({{< ref "single-total.md" >}}) will explanation the first of our Active Patterns, Single Total.