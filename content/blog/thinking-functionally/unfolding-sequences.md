+++
date = "2013-02-07T00:00:00Z"
title = "Unfolding Sequences"
tags = ["functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png"
+++


In my last post I worked through an example that finds the range of numbers that sum to a target value, or gets as close as possible without exceeding the target. I mentioned that the solution felt a little too like the loopy code I would have written in non-functional languages. I felt that there might be a more “functional” way of solving the problem, but I didn’t know what it was.

Ross McKinlay kindly added a comment to point out how the problem could be solved using the ‘unfold’ feature of sequences. When I first looked at the example, I felt like I often do when I see unfamiliar F# code. Utterly confused.

I eventually figured it out and thought it might be worth a further post. Let’s go back to where we left the code at the end of the last post.

{{< highlight fsharp "style=tango" >}}
let rec DoSumTo t currentNum currentSum =
    let nextNum = currentNum + 1
    match currentSum + nextNum > t with
    | false -> DoSumTo t nextNum (currentSum + nextNum)
    | true -> [1..currentNum]

let SumTo target =
    DoSumTo target 1 1
 
> SumTo 28;;
11
val it : int list = [1; 2; 3; 4; 5; 6; 7]
{{< / highlight >}}

Let me say at the start, there’s a bug in this code. If you try SumTo 0, you get a wrong answer. We’ll address that bug in this post.

Here’s the alternative “unfold” solution that Ross provided.

{{< highlight fsharp "style=tango" >}}
let sumTo target =
    Seq.unfold(function
    | (current,total) when total >= target -> None
    | (current,total) -> Some(current,(current+1,total+current))) (1,0) 
{{< / highlight >}}

Ross’ code has some issues too. For example…

{{< highlight fsharp "style=tango" >}}
> sumTo 2 |> ToList;;
val it : int list = [1; 2]
{{< / highlight >}}

In this case the function returns a list of numbers that sum to more than the target. No biggie, a misplaced ‘=’, an easy fix. That aside, I really like the Seq.unfold feature. So, let’s step back for a second and explain some terminology, then we’ll continue.

The following is a very quick, incomplete, and probably inaccurate description of Ranges, Lists and Sequences. It will be just enough to allow you to understand the rest of the post, but I’d urge you to read up on these things yourself for a fuller picture.

Range
-----
A Range Expression is simply a notation for defining a series of numbers from a low value to a high value. By default the increment is 1, but the Range notation also allows us to define a step size. For the purpose of this blog post we’re interested in using Range expressions to define the contents of Lists and Sequences.

List
----
A List is an ordered series of elements. Lists are immutable, and finite. To put it bluntly, lists actually contain stuff. All the elements in a list will be of the same type, but that can be pretty much anything. Numbers, words, tuples, types, or even other lists.

There are numerous ways of specifying the contents of a list. You can explicitly say what it contains…

{{< highlight fsharp "style=tango" >}}
let explicitList = [ 1; 2; 3 ]
{{< / highlight >}}

You can also use a Range expression to define the contents of a list.

{{< highlight fsharp "style=tango" >}}
let rangeList = [ 1 .. 10 ]
{{< / highlight >}}

If you want to change the increment size you can. The following range runs from 0 to 100 in steps of 5.

{{< highlight fsharp "style=tango" >}}
let skipList = [ 0..5..100 ]
{{< / highlight >}}

Sequences
---------
A Sequence is similar to a list. We can enumerate the items in the sequence, apply aggregate functions, etc. Sequences can be created using Range Expressions in much the same way that Lists can.

{{< highlight fsharp "style=tango" >}}
let rangeSequence = {1..10}
{{< / highlight >}}

Note, that the only difference from Lists is that we use braces {} instead of the square brackets [].

The killer feature of Sequence is that the terms of the sequence are generated on an “as needed” basis. So, if a sequence defines a potential range of 1,000,000 items, but a particular function only accesses the first 10 terms, then only the 10 terms are generated. This allows for potentially infinite sequences.

Unfolding Sequences
-------------------
And, with that very quick primer, we go back to the code from Ross and another much more interesting way to generate a Sequence, Unfolding.

Unfolding a sequence is actually a very simple notion. We use a function to generate the next term in the sequence based on the current term. An example is worth a thousand words, so let’s have one.

{{< highlight fsharp "style=tango, hl_lines=3" >}}
let TwoAtATimeFrom n =
    n
    |> Seq.unfold(function x -> Some(x, x+2))
 
val TwoAtATimeFrom : int -> seq<int>
 
> TwoAtATimeFrom 11;;
val it : seq<int> = seq [11; 13; 15; 17; ...]
{{< / highlight >}}

The function accepts a number and pipes it into our unfold function. This value n is used only as the starting point for the sequence. Every subsequent term is calculated based on the previous term.

The key line of code is the unfold function, highlighted above. The result of this line of code is a sequence that starts at 11 (the value of n), and increases in steps of 2. But how? Let’s tear it apart.

It’s the ‘unfold’ method from the Seq module, so this bit is obvious enough.

{{< highlight fsharp "style=tango" >}}
Seq.unfold(...)
{{< / highlight >}}

The interesting stuff is what goes on between those parenthesis.

{{< highlight fsharp "style=tango" >}}
function x -> Some(x, x+2)
{{< / highlight >}}

We pass a function to unfold that accepts a number and generates the next number. To do that you might think (as I did) that it would be a simple function along the lines of

{{< highlight fsharp "style=tango" >}}
function x -> x + 2
{{< / highlight >}}

But, the function actually contains ‘Some(x, x+2)’.
Why the ‘Some’ why both x and x + 2?

As with all things, it’s simple when you understand it, and it’s difficult until then.

‘Some’ indicates that we’re generating an Option value. By this I mean an item that may have a value, or may have none. In C# we’d call these Nullable. As long as the function produces some value the unfolding will continue. If it produces a ‘None’ value, the unfolding stops. Since the example above never produces a ‘None’ value, it is an infinite sequence.

Let’s modify it so there’s an upper limit of 1000.

{{< highlight fsharp "style=tango" >}}
let TwoAtATimeFrom n =
    n |> Seq.unfold(function
    | x when x > 1000 -> None 
    | x -> Some(x, x+2))
{{< / highlight >}}

We start with the value n, but now our unfold function has two options when generating the next term. If the term is greater than 1000, the function returns None, which stops the Sequence unfolding any further. As long as we’re less than 1000, new terms will be created.

Before we move on, let’s look at that “Some” syntax.

{{< highlight fsharp "style=tango" >}}
x -> Some(x, x+2))
{{< / highlight >}}

This says that as the sequence unfolds we take the value x (the first x), we add that value to the sequence (the second x) and then we generate the next value in the sequence by adding two to x (the x + 2). To really make this clear, lets generate another sequence.

{{< highlight fsharp "style=tango" >}}
let SeqFromTuple (x, y) =
    (x, y) |> Seq.unfold(function
    (x, y) -> Some(y, (y, x+y)))
{{< / highlight >}}

This function accepts a tuple, which is used as the starting value of the sequence that will be unfolded. As there is no provision to encounter a ‘None’ value this sequence has no set upper limit.

Each iteration creates a new tuple using the following transformation (x, y) -> (y, x+y). The following shows how that might look.

{{< highlight fsharp "style=tango" >}}
(1, 1) (1, 2) (2, 3), (3, 5), …
{{< / highlight >}}

While these tuples are created, this does not reflect the actual sequence that is produced by the code. If you look at the first parameter of the ‘Some’, you’ll see it’s just ‘y’. So, while the unfold function produces a new tuple with each step, only the ‘y’ part of the tuple is actually realised into the Sequence. So the sequence actually looks like this.

{{< highlight fsharp "style=tango" >}}
> SeqFromTuple (1,1);;
val it : seq<int> = seq [1; 2; 3; 5; ...]
{{< / highlight >}}

Before I finish let me return to the concept of Option types. These are used in situations where a value may or may not exist. An option will either have “Some” value or “None”. We’ve seen above how None can be used to flag the end of a process of unfolding a sequence. That’s not really a great example of the power of Option types.

Let’s go back to the exercise that started all of this. Given a number, find the range of numbers that sums to the target value. And recognising that not all values can be summed to exactly, we get as close as possible without exceeding the target.

Now, having learned about Option types we can improve this.

Given a number, find the range of digits that sum to the target number, and where no range sums exactly to the target, the function should indicate this by returning None.

Here’s the code.

{{< highlight fsharp "style=tango" >}}
let ThatSumClosestTo x =
    (0, 0)
    |> Seq.unfold(function
        (total, num)-> if (total + num > x) 
                       then None 
                       else Some(num,(total + num, num + 1)))
 
let ThatSumsTo x =
    let closest = 
        ThatSumClosestTo x
        |> Seq.toList
    if closest |> List.sum = x 
    then Some(closest)
    else None
 
val ThatSumClosestTo : int -> seq<int>
val ThatSumsTo : int -> int list option
 
> 28 |> ThatSumsTo;;
20
val it : int list option = Some [0; 1; 2; 3; 4; 5; 6; 7]
21
> 29 |> ThatSumsTo;;
22
val it : int list option = None
{{< / highlight >}}

Attempting to find a range of numbers, starting at 0 that sums to 28, gives 1 to 7, but for 29 there is no range of numbers that sums exactly, and so, the function indicates this by returning None.