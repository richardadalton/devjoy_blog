+++
date = "2014-08-09T00:00:00Z"
title = "Partial Application"
tags = ["functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png" 
+++

This post was supposed to be about Partial Active Patterns, but before we get to that, I want to take a small diversion to cover Partial Application of Active Patterns (which is a completely different thing). Confused? Don’t worry. Read on.

### Partial Application
I’ve described Partial Application in detail here and here, so I’m going to assume that you know how it works for regular functions. Please read those two posts if you are in any doubt.

Partial Application was raised in a question from Shawn Martin.

> what might be the best way(s) to turn SpecialCharacterCount in this post into the hard-coded SpecialCharacterCount from the previous post? In other words, do partial application.

Coincidentally Anthony Brown (@bruinbrown93) had tweeted on the same topic.

> You can also partially apply your active patterns. 
{{< highlight fsharp "style=tango" >}}
E.g. let (|ExclamationMarkCount|) = (|SpecialCharacterCount|) “!”
{{< / highlight >}}

So, let’s cover partial application of active patterns.

Recall our active pattern for counting occurrences of characters in a string.

{{< highlight fsharp "style=tango" >}}
let (|SpecialCharacterCount|) (characters: string) (str: string) =
    str.ToCharArray()
    |> Array.filter (fun c -> characters.Contains(c.ToString()))
    |> Array.length
{{< / highlight >}}

In addition to the parameter ‘str’ for the string that we’ll search, we have a parameter ‘characters’ containing the characters that we’ll search for. We can use this, and the magic of partial application to create other Active Patterns that have the characters baked in, so we don’t need to provide them as an extra argument.

{{< highlight fsharp "style=tango" >}}
let (|CountDollars|) = (|SpecialCharacterCount|) “$”
{{< / highlight >}}

We can use CountDollars just like any Active Pattern, but it’s hardwired to search for the ‘$’.

{{< highlight fsharp "style=tango" >}}
let describeString (s: string) =
    match s with
    | CountDollars d when d > 5 -> "%d is more than 5 Bucks"
    | CountDollars 4 -> "That's 4 Bucks"
    | CountDollars d -> "Just %d Dollars" d 
{{< / highlight >}}

Creating more patterns that search for different characters is trivial

{{< highlight fsharp "style=tango" >}}
let (|CountStars|) = (|SpecialCharacterCount|) "*"
{{< / highlight >}}

Now instead of counting dollars, we’ll be counting stars.
Yeah, I went there. Don’t judge me.

And, of course the argument to the original Active Pattern can contain multiple characters, and that still applies.

{{< highlight fsharp "style=tango" >}}
let (|CountCurrencies|) = (|SpecialCharacterCount|) "£$"
{{< / highlight >}}

We can mix and match our original and newly minted patterns.

{{< highlight fsharp "style=tango" >}}
let describeString (str: string) =
    match str with
    | CountCurrency c when c > 3 -> "More than three currency symbols"
    | CountDollars d when d > 0 -> "There's at least one Dollar"
    | CountStars 2 -> "There are exactly two Stars "
    | SpecialCharacterCount "!%^&" s -> sprintf "I found %d special characters" s
    | _ -> str 
{{< / highlight >}}

### Patterns
The second topic I wanted to touch on before moving forward is Patterns. In the first post in the series I gave an introduction to Pattern Matching which has served us well in understanding the examples. It was however just the briefest of introductions, let’s dig a little deeper.

In the (|IsValid|) active pattern in this post I quietly slipped in something a little unusual. I used multiple Patterns within the same clause.

{{< highlight fsharp "style=tango" >}}
| UpperCaseCount u & LowerCaseCount l & SpecialCharacterCount s -> ...
{{< / highlight >}}

For that clause to match all three of those patterns need to match. This makes sense, the ‘&’ represents ‘and’.

All of the patterns that you can use are described here so I won’t go into each in detail. I will briefly mention the ‘or’ pattern denoted by the ‘|’ operator, to give you a sense of what these patterns can do.

Take a look at this function.

{{< highlight fsharp "style=tango" >}}
let describeString (str: string) =
    match str with
    | CountDollars c | CountStars c when c > 5 -> sprintf "%d is more than 5 Dollars or Stars" c
    | _ -> str 
{{< / highlight >}}

If either CountDollars or CountStars returns a count of more than 5 then the first clause will match. If CountDollars returns more than 5 then c will hold it’s value. If not then there’s still the chance that CountStars will set the value of c. If neither are above 5 the clause in it’s entirety will fail. Note we’re not talking about the total of Dollars and Stars exceeding 5. Either Dollars or Stars must exceed 5 in it’s own right.

Literals can be used on both sides of the ‘|’ operator, but if a variable is used, it must be the same on both sides.

{{< highlight fsharp "style=tango" >}}
let describeString (str: string) =
    match str with
    | CountDollars c | CountStars c when c > 5 -> sprintf "%d Dollars or Stars, that's more than 5" c
    | CountDollars 2 | CountStars 10 -> "2 Dollars and 10 Stars"
    | _ -> str 
{{< / highlight >}}

The following is not valid for the ‘or’ pattern.

{{< highlight fsharp "style=tango" >}}
| CountDollars 2 | CountStars s -> sprintf "2 Dollars and %s Stars" s
{{< / highlight >}}

I’ll leave you to play with the other patterns. In the [next post]({{< ref "single-partial.md" >}}) we’ll get back to describing the different forms of Active Pattern.