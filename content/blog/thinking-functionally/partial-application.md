+++
date = "2013-01-27T00:00:00Z"
title = "Partial Application"
tags = ["functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png"
series = ["Thinking Functionally"]
series_weight = 01
+++

Warning, novice functional thinker here. If you know your stuff, what follows may cause distress.

I was messing with F# last night and I got a gentle reminder that I’m still a long way from thinking functionally, it still takes a lot of effort.

I started with this

{{< highlight fsharp "style=tango" >}}
let evens r = List.filter (fun x -> x % 2 = 0) r
> evens [0..10];;

val it : int list = [0; 2; 4; 6; 8; 10]
{{< / highlight >}}

Simple enough. Even numbers are multiples of 2. Rekindling childhood memories of typing code from books by Tim Hartnell into a ZX81, I did what I would have done then. I tried to make the code do something slightly more complicated. What about a function that can find multiples of any number, not just 2.

So, how would I write this next function?

{{< highlight fsharp "style=tango" >}}
> multiplesOf 3 [0..10];;
{{< / highlight >}}

I played with the ‘even’ function for a while. Clearly I needed to replace the 2 with a variable, but no matter what I tried I couldn’t find a way of getting a variable into the filter predicate.

I was missing the point. The filter predicate takes one argument, a member of the list being filtered. There is, to the best of my knowledge, no way of adding extra parameters. You need to take a different (more functional) approach.

Partially Applying Functions
----------------------------
What we need to do is create a function that accepts an item from the list and checks if it’s a multiple of a variable, without passing that variable into the function. In other words, we need to lock the variable in when the function is created. What we’re talking about here is partial application, which is something I thought I understood, but as with any new concept, reading about it is one thing, having it pop into your mind automatically when you need it is another matter entirely.

When the partial application idea had made it’s way into my mind coming up with something that works was relatively easy (that’s progress I suppose).

{{< highlight fsharp "style=tango" >}}
let multipleOf x y = y % x = 0;

let multiplesOf m r =
    let multipleOfm = multipleOf m;
    List.filter (fun x -> multipleOfm x) r;
{{< /highlight >}}

The multipleOf function takes care of figuring out if any number is a multiple of another. It takes two parameters for obvious reasons. But through the magic of partial application if we call multipleOf passing it only a value for m, it will return a function that accepts the remaining one argument, which it checks against m to see if it’s a multiple.

<blockquote>...in imperative programming we focus on writing functions, in functional programming we will often create the function that we need at runtime, and partial application is one tool at our disposal for doing that.</blockquote>

In other words, our partially applied version ‘multiplesOfm’ will work as the predicate filter, allowing us to create the function that all this started with.

That all felt like a tiny breakthrough in ‘functional thinking’ the understanding that in imperative programming we focus on writing functions, in functional programming we will often create the function that we need at runtime, and partial application is one tool at our disposal for doing that.

To really lock this idea in, let’s play with it a little more. Let’s rewrite our ‘evens’ function, using the ‘multipleOf’ function.

{{< highlight fsharp "style=tango" >}}
let multipleOf x y = y % x = 0;

let even = multipleOf 2;

> even 4;;
val it : bool = true
{{< / highlight >}}

It’s worth looking at why this all works. Something I don’t have time to do right now, maybe another time, but if you are trying to learn how to “Think Functionally” you could do worse than get a solid grasp of partial application.