+++
date = "2014-08-18T00:00:00Z"
title = "Choices And Nesting"
tags = ["functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png" 
aliases = [
    "/2014/08/active-patterns-choices-and-nesting/",
]
+++

And so, we arrive at the last post of the series. I’ll show you the F# ‘Choice’ type and show how it relates to active patterns. I’ll explain how to add additional parameters to a multi-case active pattern, and introduce some complex pattern matching using nested active patterns.

### Look Ma! No Params
In the previous post I mentioned that Multi-Case active patterns can’t accept additional arguments beyond the one that all active patterns must accept. Here’s what the code might look like if they could.

{{< highlight fsharp "style=tango" >}}
let (|Smaller|Equal|Bigger|) (compareWith: int) (n: int) =
    if n < compareWith then Smaller
    elif n > compareWith then Bigger
    else Equal

let DescribeNumber (n: int) =
    match n with
    | Smaller 10 -> "Small"
    | Bigger 10 -> "Bigger"
    | Equal 10 -> "Equal"
{{< / highlight >}}

On the face of it that looks like reasonable code. To see why it’s invalid, let’s change the DescribeNumber function slightly.

{{< highlight fsharp "style=tango" >}}
let DescribeNumber (n: int) =
    match n with
    | Smaller 5 -> "Small"
    | Bigger 10 -> "Bigger"
    | Equal 10 -> "Equal"
{{< / highlight >}}

Assuming we could do this, what should happen if we pass the number 7 to DescribeNumber?

The first clause won’t match, because 7 is not smaller than 5. The second and third clauses won’t match either because 7 isn’t bigger than or equal to 10. So, we’ve taken a multi-case pattern, where they whole point is the completeness of the pattern matching, and we’ve broken that completeness by parameterizing each clause separately.

If you try this code out you’ll find that the error isn’t in the Pattern Recognizer, it’s in DescribeNumber. In other words, it’s fine to define a multi-case active pattern, as long as you don’t try to pattern match on it. That offers us a clue as to how to proceed.

{{< highlight fsharp "style=tango" >}}
let (|SmallerThan10|EqualTo10|BiggerThan10|) = (|Smaller|Equal|Bigger|) 10
{{< / highlight >}}

If we partially apply the Active Pattern, locking in a value for the additional parameter, we’re back to a multi-case active pattern with one parameter. And our pattern match now looks like this.

{{< highlight fsharp "style=tango" >}}
let DescribeNumber (n: int) =
    match n with
    | SmallerThan10 -> "Small"
    | BiggerThan10 -> "Bigger"
    | EqualTo10 -> "Equal"
{{< / highlight >}}

The original function has the values Smaller, Equal and Bigger and the partially applied version has SmallerThan10, EqualTo10 and BiggerThan10. Which begs the question, how does one map to the other? Is it just their positions?

Well, yes, that’s exactly how it works, but for an interesting reason that explains a lot about Multi-Case Active Patterns. Go back and take a look at the signature of the original active pattern.

{{< highlight fsharp "style=tango" >}}
val (|Smaller|Equal|Bigger|) :
  compareWith:int -> n:int -> Choice<unit,unit,unit>
{{< / highlight >}}

It returns a Choice

### Give me Choice
Just as the Option type allowed us to write functions that can return either ‘Some value’ or ‘None’. The Choice type allows us to return one of up to seven values (remember that seven item limit on multi-case active patterns?).

Forget about Active Patterns for a moment and look at this plain old function.

{{< highlight fsharp "style=tango" >}}
let Divide (by: int) (this: int) =
    if by = 0 then Choice1Of3 "You can't divide by zero, are you mad?"
    elif this % by = 0 then Choice2Of3 (this/by)
    else Choice3Of3 (decimal this/decimal by)
{{< / highlight >}}

It’s type is as follows

{{< highlight fsharp "style=tango" >}}
val Divide : by:int -> this:int -> Choice<string,int,decimal>
{{< / highlight >}}

Depending on the numbers you try to divide, this function will return one of three choices. A string (an error message for divide by zero), an int (numbers divide evenly), or a decimal (doesn’t divide evenly.)
We can pattern match on this function just like we do with Multi-Choice Active Patterns.

{{< highlight fsharp "style=tango" >}}
let DescribeDivide (this: int) (by: int) =
    match Divide this by with
    | Choice1Of3 error -> error
    | Choice2Of3 n -> sprintf "%d divides evenly by %d and the result is %d" this by n
    | Choice3Of3 n -> sprintf "%d doesn't divide evenly by %d and the result is %f" this by n
{{< / highlight >}}

It may be starting to dawn on you now that the Multi-Case Active Pattern is a way of assigning names to those choices. In fact, if you take any function that takes a single argument and returns a Choice you can turn it into a Multi-Case Active Pattern, and call the various possibilities anything you like.

The following example partially applies the Divide function, the result is a Choice which can be turned into a multi-choice active pattern, and used immediately in a pattern match, all within a single function.

{{< highlight fsharp "style=tango" >}}
let DescribeDivide (by: int) (this: int) =
    let (|ByZero|Evenly|NotEvenly|) = Divide by
    match this with
    | ByZero error -> error
    | Evenly n -> sprintf "%d divides evenly by %d and the result is %d" this by n
    | NotEvenly n -> sprintf "%d doesn't divide evenly by %d and the result is %f" this by n
{{< / highlight >}}

With all that in mind look again at the partially applied Multi-Case Active Pattern from above.

{{< highlight fsharp "style=tango" >}}
let (|SmallerThan10|EqualTo10|BiggerThan10|) = (|Smaller|Equal|Bigger|) 10
{{< / highlight >}}

The signature of the original Active Pattern is

{{< highlight fsharp "style=tango" >}}
compareWith:int -> n:int -> Choice<unit,unit,unit>
{{< / highlight >}}

Partially applying it we get a new function with the following signature

{{< highlight fsharp "style=tango" >}}
int -> Choice<unit,unit,unit>
{{< / highlight >}}

One input parameter, and the out put is a Choice Of 3, so we can assign this to any Multi-Case Active Pattern name as long as it has 3 cases, in our example (|SmallerThan10|EqualTo10|BiggerThan10|).

### Code Quotations
For this final example I need a reasonably elaborate domain model. Instead of creating a phony one, I’m going to use F# itself. Code Quotations provide a model for working with F# code as Data. The point of this example is to convey that even after 8 posts explaining Active Patterns in detail, there is still enormous scope to combine and use Active Patterns in interesting ways.

{{< highlight fsharp "style=tango" >}}
let model = <@ fun (x: int) -> x * x @>
val model : Expr<(int -> int)> = Lambda (x, Call (None, op_Multiply, [x, x]))
{{< / highlight >}}

The quoted expression is a lambda that takes an integer x and squares it. The quotation determines that we have an int->int expression. Without getting too bogged down in the syntax, you should be able to see that it’s a Lambda that accepts a variable x and calls a multiply operation, passing two arguments, both of which are x.

OK, now you’re up to speed on Code Quotations, or as up to speed as you’re going to get from this post. It might be worth having a quick read of this page before continuing.

Here are some Namespaces we’ll make use of.

{{< highlight fsharp "style=tango" >}}
open Microsoft.FSharp.Quotations
open Microsoft.FSharp.Quotations.Patterns
open System.Reflection
{{< / highlight >}}

That Patterns namespace will come in handy. Code Quotations come with a set of ready made Active Patterns. The following example uses two supplied patterns. ‘Call’ matches method calls, and ‘Value’ matches values. Both are Single Partial Active Patterns.

{{< highlight fsharp "style=tango" >}}
let rec Describe (expr: Expr) =
    match expr with
    | Call (_,f, left :: right :: []) -> sprintf "%s %s %s" (Describe left) f.Name (Describe right)
    | Value (v, t) when t.Name = "Int32" -> sprintf "%A" v
    | _ -> "?"
{{< / highlight >}}

This function matches on method calls and values. It will fail to match on any other expression. The Call active pattern returns a three part tuple. We don’t care about the first part, the second part is the function that is called, which we bind to the name f. The third part of the tuple contains the list of arguments. We’ll match on calls that have exactly two arguments, and we bind them to the names left and right.

We’ll also match on values. The Value Active Pattern returns a tuple containing the value and the type. We bind the names v and t to those respectively, and we’ll only match if the type is ‘Int32’.

We add printing of the results and we get the following.

{{< highlight fsharp "style=tango" >}}
let desc = Describe <@ 2 - 1 * 5 - 3 @>
val desc : string = "2 op_Subtraction 1 op_Multiply 5 op_Subtraction 3"
{{< / highlight >}}

OK, it ain’t pretty, but hopefully you can see how the Active Patterns are working. If we throw something other than integers at it, we won’t match the values.

{{< highlight fsharp "style=tango" >}}
let desc = Describe <@ 2.0 - 1.0 * 5.0 - 3.0 @>
{{< / highlight >}}

val desc : string = "? op_Subtraction ? op_Multiply ? op_Subtraction ?"

### Nesting
Now let’s have some fun. We’ll need to create two Active Patterns.

{{< highlight fsharp "style=tango" >}}
let (|MethodWithName|_|) (s:string) (m: MethodInfo) =
    if s = m.Name then Some() else None

let (|TypeWithName|_|) (s:string) (t: System.Type) =
    if s = t.Name then Some() else None
{{< / highlight >}}

The first pattern accepts a string and a MethodInfo and matches if the method has the name we provide. The second does the same, but with the name of a Type.

Here’s where things get interesting, the second element of the Tuple returned by the Call Active Pattern is a MethodInfo, which means we can pass it to this new Active Pattern. Like this.

{{< highlight fsharp "style=tango" >}}
let rec Describe (expr: Expr) =
    match expr with
    | Call (_,MethodWithName "op_Multiply", [left; right]) -> 
        sprintf "%s %s %s" (Describe left) "*" (Describe right)
    | Call (_,MethodWithName "op_Subtraction", [left; right]) -> 
        sprintf "%s %s %s" (Describe left) "-" (Describe right)
    | Value (v, TypeWithName "Int32") -> 
        sprintf "%A" v
    | _ -> "?"
{{< / highlight >}}

We’ve nested a match on specific method names within matches on method Calls. This will now only match on calls to op_Multiply and op_Subtract. Knowing this we can describe the methods with their operators ‘*’ and ‘-‘ rather than the ugly full method names.

I did the same with the ‘TypeWithName’ active pattern and removed the ‘When’ clause in the process.
Let that sink in. You can use an active pattern to match on values returned from another active pattern. You don’t need a whole separate ‘match .. with’ expression.

I’ve only scratched the surface of nested types, for a much more detailed discussion this talk by Ross McKinlay is heavy going in places but really showcases the power of Active Patterns.

### That’s all folks
That’s it for this series. I hope you found it worthwhile. We’ve covered an enormous amount of information. If you’ve managed to work through the entire series of posts you should be fully comfortable with Active Patterns. All that remains is the put them to good use.

Thanks for reading.