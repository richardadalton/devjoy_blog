+++
date = "2015-01-24T17:43:00Z"
title = "Avoid Branches In Your Code"
abstract="An if statement creates a branch in your code and increases it's complexity. This post looks at a less obvious kind of branch."
tags = ["code"]
categories = ["how to"]
banner = "img/banners/t-junction-sign.png"
aliases = [
    "/2015/01/minimize-mental-overhead-for-code-readers/",
]
+++

This week at our FunctionalKats meetup in Dublin, we tackled a simple programming task the Luhn checksum algorithm. The challenge was to try and make the code readable.

I rattled off some code that worked, but I wasn’t at all happy with it. It implemented the problem as described. Partitioning the numbers into two groups and dealing with each group in turn.

{{< highlight fsharp "style=tango" >}}
let Luhn s =
    let double x = x*2

    let sumDigits = function
        | 18 -> 9
        | n when n > 9 -> n%9
        | n -> n

    let odds, evens =
        s
        |> Seq.map (string >> int)
        |> Array.ofSeq
        |> Array.rev
        |> Array.partitioni (fun i -> i%2 = 0)

    let oddSum = Array.sum odds
    let evenSum =
        evens
        |> Array.map double
        |> Array.map sumDigits
        |> Array.sum
    (oddSum + evenSum) % 10 = 0
{{< /highlight >}}

The Array.partitioni function doesn’t exist, I had to write that too.

{{< highlight fsharp "style=tango" >}}
module Array =
    let partitioni f a =
        let first, second =
            a
            |> Array.mapi (fun i x -> f i, x)
            |> Array.partition (fun (i, _) -> i)
        (Array.map snd first, Array.map snd second)
{{< /highlight >}}

I didn’t like the idea of partitioning the numbers. That may match the problem description but I think we can do better. I like functions that take in something and transforms it, perhaps through many transformations, but without splitting or branching the logic.

If a function splits the data and does separate transformations, there’s mental overhead involved. Every variable in a function is an extra concept for the reader to understand.

My second attempt tried to rectify this

{{< highlight fsharp "style=tango" >}}
let Luhn s =
    let double x = x*2

    let sumDigits = function
        | 18 -> 9
        | n when n > 9 -> n%9
        | n -> n

    let mapOdd = id
    let mapEven = (double >> sumDigits)
    let mapNumber i = if i%2=0 then mapOdd else mapEven

    let total =
        s
        |> Seq.map (string >> int)
        |> Array.ofSeq
        |> Array.rev
        |> Array.mapi mapNumber
        |> Array.sum

    total % 10 = 0
{{< /highlight >}}

Instead of partitioning the data we map the entire array. The function used in the mapping is different for odd and even positions. The mapNumber function handles the toggling of the two functions.

This is better, but it’s still not right. Those mapOdd and mapEven functions, that IF expression selecting between them. It still feels like two trains of thought. We can do better.

{{< highlight fsharp "style=tango" >}}
let sumDigits n = Math.DivRem(n, 10) ||> (+)
let isMultipleOf x y = y % x = 0

let Luhn: string->bool =
    Seq.map (string >> int) 
    >> Seq.rev
    >> Seq.map2 (*) (Seq.cycle (seq[1;2])) 
    >> Seq.map sumDigits
    >> Seq.sum
    >> isMultipleOf 10
{{< /highlight >}}

Now we’re getting much closer to the ideal. The function accepts a string, produces a bool. It gets from string to bool through series of simple transformations. No IF expressions, no splitting of the numbers into groups, no toggling of functions.

The trick lies in generating a cycle of 1’s and 2’s. When you multiply a sequence by 1,2,1,2,1,2…. You double every second item in the sequence. With that done, the rest of the function falls into place.

There are a few other subtle improvements. The sumDigits function is simpler thanks to the use of Map.DivRem and ||>.  The Luhn function itself is a composition of transformations. I also created the Seq.cycle and Seq.rev.

{{< highlight fsharp "style=tango" >}}
module Seq =
    let rec cycle s = seq { yield! s; yield! cycle s}

    let rev s = s |> Array.ofSeq |> Array.rev |> Seq.ofArray
{{< /highlight >}}

I’m happy with the function as it stands. The ideas I wanted to get across with this post are

Don’t let the description of a problem constrain your thinking. It’s often possible to find solutions that are simpler than the problem would imply.

Try to reduce the moving parts, the “concepts” in your functions. Aim for functions that read straight through. A reader should have to push as little as possible onto their mental stack.

Try to avoid splitting your data and working on different subsections of it. A function should only accept the data it needs to work on. Keeping that data together as you transform it isn’t always possible. If you can achieve it, your code will usually be simpler.