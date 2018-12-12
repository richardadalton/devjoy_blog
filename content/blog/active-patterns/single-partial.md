+++
date = "2014-08-11T00:00:00Z"
title = "Single Partial (|A|_|)"
tags = ["functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png" 
series = ["F-Sharp Active Patterns"]
series_weight = 04
+++

I’ve referred to all of the Active Patterns we have seen so far in this series as ‘Single Total’. It’s time to look at the distinction between ‘Total’ and ‘Partial’ Active Patterns.

To understand Partial Active Patterns you need to have some understanding of Option Types’. If they are new to you, I’d encourage you to read up on them before continuing. A great place for reading about this, and F# generally is FSharpForFunAndProfit.

When we write a function to count the upper case characters in a string, there is always an answer, even if that answer is 0. Not all functions are so clear cut. If we write a function to parse a string into a number, it is possible to provide strings that simply can’t be parsed.

{{< highlight fsharp "style=tango" >}}
ParseNumber(“21”)	// OK
ParseNumber(“Tango Foxtrot”)	// What can we do with this?
{{< / highlight >}}

All of the examples we’ve seen so far have involved Active Patterns that are just transformation functions. The match expression uses guards, literal values, variables and wildcards to decide how it will match on the returned value.

Partial Active Patterns take back some of that control. A ParseNumber active pattern can decide that it won’t match a value regardless of how the match expression is constructed. It does this by returning an Option Type. When it returns ‘Some Value’ the match expression will deal with that value just as before, but if the Active Expression returns ‘None’ then there is no match and the value falls through and must be handled by a different pattern.

Since we’ve been talking about parsing numbers, let’s look at an example.

{{< highlight fsharp "style=tango" >}}
let (|Number|_|) (str: string) =
    match Int32.TryParse(str) with
    | (true, n) -> Some(n)
    | false, _ -> None
{{< / highlight >}}

The Active Pattern name now includes an extra component. The underscore (wildcard) indicates that this is a partial Active Pattern. Not every string that is passed to this pattern can be turned into a Number. We can use this in a match expression as follows.

{{< highlight fsharp "style=tango" >}}
let IsItANumber (str: string) =
    match str with
    | Number n -> sprintf "That was number %d" n
    | _ -> sprintf "Not a number"
{{< / highlight >}}

Option Types are still valid types. You can return an option from a SingleTotal active pattern.

{{< highlight fsharp "style=tango" >}}
let (|Number|) (str: string) =
    match Int32.TryParse(str) with
    | (true, n) -> Some(n)
    | false, _ -> None

let IsItANumber (str: string) =
    match str with
    | Number (Some n) -> sprintf "That was number %d" n
    | _ -> sprintf "Not a number"
{{< / highlight >}}

The remainder of the Active Pattern is identical, but, because we used the SingleTotal form of the Active Pattern we have to unpack the value n from the option type using the ‘Some’ keyword. By using the Partial Form of the active pattern we get that for free.

I’ll be honest I find that a little confusing because the signatures of both forms of the pattern are the same (string -> int option). Don’t worry too much about it (I don’t), if you are writing Single case Active Patterns that can’t match all possible inputs then use the Partial form.

### The Infamous FizzBuzz
Let’s take another example of a Total Active Pattern and convert it to a Partial, and see what happens. Here is the infamous FizzBuzz

{{< highlight fsharp "style=tango" >}}
let (|Multiple|) m n =
    if n%m=0 then true
    else false

let (|Fizz|) = (|Multiple|) 3
let (|Buzz|) = (|Multiple|) 5

let FizzBuzz n =
    match n with
    | Fizz true & Buzz true -> "FizzBuzz"
    | Fizz true -> "Fizz"
    | Buzz true -> "Buzz"
    | _ -> n.ToString()
{{< / highlight >}}

The Active Pattern (|Multiple|) returns a boolean indicating whether parameter ‘n’ is a multiple of ‘m’. Because it returns true or false, our match expression needs to specify those literals in order to catch the various cases.

Here’s the same code rewritten using Partials

{{< highlight fsharp "style=tango" >}}
let (|Multiple|_|) m n =
    if n%m=0 then Some Multiple
    else None

let (|Fizz|_|) = (|Multiple|_|) 3
let (|Buzz|_|) = (|Multiple|_|) 5

let FizzBuzz n =
    match n with
    | Fizz & Buzz -> "FizzBuzz"
    | Fizz -> "Fizz"
    | Buzz -> "Buzz"
    | _ -> n.ToString()
{{< / highlight >}}

The booleans are gone, the Active Patterns now return Multiple, Fizz, Buzz and None leading to a cleaner match expression. Both those examples also use Currying and the ‘&’ pattern from the last post.

### An Expression Evaluator
Let’s finish this post with one last example, a little more ambitious this time. We’ll start with a match expression and see if we can figure out the Active Patterns to make the magic happen.

{{< highlight fsharp "style=tango" >}}
let rec Eval str =
    match str with
    | Number x -> x
    | Addition (l, r) -> Eval(l) + Eval(r)
    | Subtraction (l, r) -> Eval(l) - Eval(r)
    | _ -> raise (System.ArgumentException("Invalid Expression " + str))
{{< / highlight >}}

This function evaluates expressions. It seems to only handle Numbers, Addition and Subtraction so we’ll need Active Patterns for each of those.

It looks like the expression is represented as a string, but the pattern matching would be basically identical even if expression were represented in another way. That’s one of the perks at coding at a higher level of abstraction.

Number looks like the example we’ve already seen, let’s do that first.

{{< highlight fsharp "style=tango" >}}
let (|Number|_|) (str: string) =
    match Int32.TryParse(str) with
    | (true, n) -> Some(n)
    | false, _ -> None
{{< / highlight >}}

Incidentally if that syntax for the TryParse function is throwing you a little, have a look at this post.

Both Addition and Subtraction accept strings and return what look like tuples representing the left and right subexpressions.

{{< highlight fsharp "style=tango" >}}
let (|Addition|_|) (str: string) =
    let SplitAt (op: string) (str: string) =
        let pos = (str.IndexOf op)
        (str.Substring(0, pos),str.Substring(pos + 1, str.Length - pos - 1))

    if str.Contains("+") then
        Some (SplitAt "+" str)
    else
        None

let (|Subtraction|_|) (str: string) =
    let SplitAt (op: string) (str: string) =
        let pos = (str.IndexOf op)
        (str.Substring(0, pos),str.Substring(pos + 1, str.Length - pos - 1))

    if str.Contains("-") then
        Some (SplitAt "-" str)
    else
        None
{{< / highlight >}}

Some code duplication there, but it will work. Both patterns check the incoming string. If it contains the right operator the patterns split the string and returns either side as the elements of a tuple. The Eval function then takes those two subexpressions, recursively evaluates them and applies the right operator to bring the two expressions back together.

The [next post]({{< ref "single-partial-with-params.md" >}}) will be very short and will show how to use parameters with Partial Active Patterns. This won’t need much explaining because it’s the same process as adding parameters to Total Active Patterns which you’ve already seen. We’ll use parameters to rewrite this expression evaluator and get rid of the duplicated code.
