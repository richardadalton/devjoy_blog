+++
date = "2014-08-07T00:00:00Z"
title = "Single Total With Params (|A|) x"
tags = ["functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png"
aliases = [
    "/2014/08/active-patterns-single-total-with-params-a-x/",
]
+++

We move on to the next in our series on Active Patterns, but this time we’re really just covering a slight modification to the Single Total pattern that we covered in the [last post]({{< ref "single-total.md" >}}).

All the same rules apply, we’re just adding the ability to add parameters to the Active Pattern.

I say ‘parameters’ but in reality I mean ‘additional parameters’. Every Active Pattern has at least one parameter. The ‘x’ in ‘match x with’ has to go somewhere.

Remember the Active Patterns we’ve seen so far are just functions. If you take away the banana clips they are identical to functions. Additional parameters are supported exactly the same way they are with regular functions.

All you need to remember is that the ‘x’ in ‘match x with’ will be passed to the last parameter of the Active Pattern. The other parameters are set by the individual lines in the Pattern Match expression.

An example might help.

Here’s the Active Pattern we used in the last post to count special characters in a string.

{{< highlight fsharp "style=tango" >}}
let (|SpecialCharacterCount|) (str: string) =
    let specialCharacters = "!£$%^"
    str.ToCharArray()
    |> Array.filter (fun c -> specialCharacters.Contains(c.ToString()))
    |> Array.length
{{< / highlight >}}

Note the special characters are hard coded. This being a function it’s simple to turn that into a parameter.

{{< highlight fsharp "style=tango" >}}
let (|SpecialCharacterCount|) (characters: string) (str: string) =
    str.ToCharArray()
    |> Array.filter (fun c -> characters.Contains(c.ToString()))
    |> Array.length
{{< / highlight >}}

You can add multiple parameters but it’s important to know how they map to the pattern matching expression that will use the Active Pattern.

Here’s code that would match against the old ‘non-parameterized’ Active Pattern

{{< highlight fsharp "style=tango" >}}
let UseThePattern (str: string) =
    match str with
    | SpecialCharacterCount c -> sprintf "That was %d special characters" c
{{< / highlight >}}

By contrast here’s the code that uses the parameterized version

{{< highlight fsharp "style=tango" >}}
let UseThePattern (str: string) =
    match str with
    | SpecialCharacterCount "!£$%^" c -> sprintf "That was %d special characters" c
{{< / highlight >}}

The ‘str’ in the match expression still gets passed to the last parameter of the Active Pattern, and the return value of the Active Pattern still gets assigned to ‘c’. In place of c you can still use literals just as in the last post.

The only real difference is that the extra arguments are placed between the name of the Active Pattern and the variable or literal value that we match against.

Here’s another sketch to try and hammer this home.

{{< figure src="https://res.cloudinary.com/devjoy/image/upload/v1537281244/devjoy_blog/SingleTotalActivePatternWithParams.jpg" title="Single Total Active Pattern With Parameters" >}}

Apologies for spelling this out in such tedious detail but if you get what’s happening here, everything else about Active Patterns will be very simple.

Oh, and that little assignment trick we saw in the last post still works, just pass the arguments needed by the Active Pattern.

{{< highlight fsharp "style=tango" >}}
let (SpecialCharacterCount "!£$%^" c) = "ABC!"
val c : int = 1
{{< / highlight >}}

That’s it for this post and for my nemesis the Single Total Active Pattern. [Next up]({{< ref "partial-application.md" >}}) we discover the wonders of Partial Active Patterns. We also get to rope in another interesting piece of F#, Option Types.