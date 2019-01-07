+++
date = "2013-02-01T00:00:00Z"
title = "Iterating, Incrementing, and Accumulating"
tags = ["functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png" 
aliases = [
    "/2013/02/learning-to-think-functionally-iterating-incrementing-and-accumulating/",
]
+++


Another F# session this evening and some more deliberate practice of Functional Thinking. To be fair, this post isn’t really about anything new. If you’ve ever used recursion, even in non-functional languages, this will be old news. If you are new to Functional Programming and/or recursion then this may be useful.

Here’s a really simple function. It accepts a number n and sums all the numbers from 1 to n.

{{< highlight fsharp "style=tango" >}}
let SumUpToValue n = [1..n] |> List.sum
{{< / highlight >}}

I create a range up to n and then it’s a simple matter to sum the numbers in the range.

What if I wanted to start from a target value, and work back to the range of numbers?

{{< highlight fsharp "style=tango" >}}
> SumTo 10;;
val it : int list = [1; 2; 3; 4]
{{< / highlight >}}

Let’s agree one little ground rule here. Not every target number can be reached exactly by summing a range. Our function will get us as close as possible to the target without exceeding it.

If I was back in Imperativeland I’d create a loop starting from 1 and add to an accumulator until I reach the target. But here in Functionalville that’s apparently not how things are done. So, how are things done? How can I implement this function without iterating over a series of numbers, summing them, and using a loop guard so I know when to stop?

It turns out I can’t. That’s not to say it can’t be done, but the most “functional” solution I could come up with is basically the same solution as the loop described above, albeit with a cunning disguise.

Here was my first attempt

{{< highlight fsharp "style=tango" >}}
let rec DoSumTo t nums =
    let currentSum = List.sum nums
    let nextNumber = nums.Head + 1
    match currentSum + nextNumber > t with
    | true -> nums
    | false -> DoSumTo t (nextNumber::nums)
 
let SumTo target =
    DoSumTo target [1]

> SumTo 21;;
13
val it : int list = [6; 5; 4; 3; 2; 1]
{{< / highlight >}}

‘DoSumTo’ is a recursive function that is passed the target value (t), and the list of numbers found so far (nums). The list of numbers provides us with two useful pieces of information. Adding one to the number at the head of he list gives us the next number that we need to add. So, the list is actually doubling up as the variable we would normally use as a loop counter.

Summing the contents of the list allows us to check if we’ve reached the target value yet, so the list is also doubling as the accumulator we would use if we were writing this using imperative code with mutable variables.

The function checks whether adding the next number to the list would cause us to exceed the target. If it would, we return the list as it currently stands, and that’s our answer. If adding the next number doesn’t exceed the target, the function recursively calls itself to try the next number.

The only reason I’m using pattern matching here rather than if..else is that as part of my practice I’m trying to avoid using If statements and While statements.

When ‘DoSumTo’ calls itself recursively it passes the target (t) unchanged, but it creates a new list consisting of the next number as the head, and the existing list of numbers as the tail. In other words, each new number is added at the head of the list.

This of course means that the resulting list of numbers is in reverse order. I don’t really care about this. The numbers are correct and reversing the list is trivial.

There is one potentially big problem with this code. Every call to ‘DoSumTo’ causes the list of numbers to be summed. That’s quite wasteful, and easily fixed. Let’s take another stab at this.

{{< highlight fsharp "style=tango" >}}
let rec DoSumTo t nums currentSum =
    let nextNumber = nums.Head + 1
    match currentSum + nextNumber > t with
    | false -> DoSumTo t (nextNumber::nums) (currentSum + nextNumber)
    | true -> nums

let SumTo target =
    DoSumTo target [1] 1
{{< / highlight >}}

We’ve eliminated the summing of the whole list for each new number, by passing the currentSum as a parameter. Each run of the function increases the currentSum for the next run creating an accumulator.

We can go further. We don’t even need to pass the full list of numbers. The function uses the head of the list to keep track of what the next number should be, but we don’t use the full list until we’ve reached the target. Manipulating a full list when we don’t need to is wasteful.

{{< highlight fsharp "style=tango" >}}
let rec DoSumTo t currentNum currentSum =
    let nextNum = currentNum + 1
    match currentSum + nextNum > t with
    | false -> DoSumTo t nextNum (currentSum + nextNum)
    | true -> [1..currentNum]
 
let SumTo target =
    DoSumTo target 1 1
{{< / highlight >}}

Now, instead of passing the full list, we just pass the number we’ve reached as we count up from 1. We also pass the target and the current sum of numbers up to that point. Now instead of using the head of the list of numbers, we just use currentNum.

The only other change is that when we reach the target number, we need to create a range for numbers from 1 to currentNum. This has the added bonus that the resulting list of numbers happens to be in the right order.

{{< highlight fsharp "style=tango" >}}
> SumTo 28;;
val it : int list = [1; 2; 3; 4; 5; 6; 7]
{{< / highlight >}}
