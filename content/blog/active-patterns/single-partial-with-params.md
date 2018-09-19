+++
date = "2014-08-11T00:00:00Z"
title = "Single Partial With Params (|A|_|) x"
tags = ["functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png" 
+++

We close out the discussion of Single Active Patterns by adding parameters to the partial active pattern. If you’ve read the post on adding parameters to the Single Total Active Pattern then there is absolutely nothing new here, it works in exactly the same way.

For that reason I’m not going to use this post to explain how to do it, I’m just going to work through an example and leave it at that.

In the last post I created some active patterns that we could use to evaluate a rudimentary arithmetic expression. There was a lot of duplicated code between the Addition and Subtraction cases.

Here’s a first stab at eliminating some of that by extracting the underlying behavior and using Partial Application. This has all been covered already in this post.

Parsing numbers is handled just like it was before, no change here.

{{< highlight fsharp "style=tango" >}}
let (|Number|_|) (str: string) =
    match Int32.TryParse(str) with
    | (true, n) -> Some(n)
    | false, _ -> None
{{< / highlight >}}

Now instead of fully implementing Addition and Subtraction independently of each other, we have a more general purpose pattern called (|BinaryOp|_|). Note that the operator (the text to look for in the expression) is an argument to this function.

{{< highlight fsharp "style=tango" >}}
let (|BinaryOp|_|) operator (str: string) =
    let SplitAt (op: string) (str: string) =
        let pos = (str.IndexOf op)
        (str.Substring(0, pos),str.Substring(pos + 1, str.Length - pos - 1))

    if str.Contains(operator) then
        Some (SplitAt operator str)
    else
        None
{{< / highlight >}}

Now creating our Addition and Subtraction patterns is simple.

{{< highlight fsharp "style=tango" >}}
let (|Addition|_|) = (|BinaryOp|_|) "+"
let (|Subtraction|_|) = (|BinaryOp|_|) "-"
{{< / highlight >}}

And the Match works just like it did before, no change here.

{{< highlight fsharp "style=tango" >}}
let rec Eval str =
    match str with
    | Number x -> x
    | Addition (l, r) -> Eval(l) + Eval(r)
    | Subtraction (l, r) -> Eval(l) - Eval(r)
    | _ -> raise (System.ArgumentException("Invalid Expression " + str))
{{< / highlight >}}

That’s all fine, but notice how our match expression needs to know how to respond to the Addition and Subtraction cases, this isn’t ideal.

{{< highlight fsharp "style=tango" >}}
| Addition (l, r) -> Eval(l) + Eval(r)
| Subtraction (l, r) -> Eval(l) - Eval(r)
{{< / highlight >}}

We could do the following.

{{< highlight fsharp "style=tango" >}}
type Operator = string * (int->int->int)
{{< / highlight >}}

The type ‘Operator’ consists of a string (the token) and a function int->int->int (I’m keeping things simple here, int only). Now our underlying BinaryOp pattern just doesn’t take a string to search for, it also takes a function to perform when that string is found.

{{< highlight fsharp "style=tango" >}}
let (|BinaryOp|_|) (operator: Operator) (str: string) =
    let token, operatorFunction = operator
    let SplitAt (op: string) (str: string) =
        let pos = (str.IndexOf op)
        (operatorFunction, str.Substring(0, pos),str.Substring(pos + 1, str.Length - pos - 1))

    if str.Contains(token) then
        Some (SplitAt token str)
    else
        None
{{< / highlight >}}

We use destructuring assignment to extract the token and the function from operator. The token is used to split the string, but the function is simply passed back from the pattern so that the calling match expression can use it.

Defining the Addition and Subtraction patterns is now slightly different, we need to include both the token and the function.

{{< highlight fsharp "style=tango" >}}
let (|Addition|_|) = (|BinaryOp|_|) ("+", (+))
let (|Subtraction|_|) = (|BinaryOp|_|) ("-", (-))
{{< / highlight >}}

As mentioned above our match expression needs to be modified to get the function back from the active pattern so that it can apply it to the left and right subexpressions. Now that both Addition and Subtraction are handled identically, we can combine them using an or ‘|’ pattern.

{{< highlight fsharp "style=tango" >}}
let rec Eval str =
    match str with
    | Number x -> x
    | Addition (f, l, r) | Subtraction (f, l, r)
            -> f (Eval(l)) (Eval(r))
    | _ -> raise (System.ArgumentException("Invalid Expression " + str))
{{< / highlight >}}

And that’s it for Single Case Active Pattern both the Total and Partial variety. We’ve covered a lot of ground. In the [next post]({{< ref "multi-case.md" >}}) we’ll move on the Active Patterns that have Multiple results. If you’ve managed to stay with me this far nothing that remains will phase you.
