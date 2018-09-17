+++
date = "2013-05-14T00:00:00Z"
title = "Memoization"
tags = ["functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png"
+++

Imagine you have a long running function that you’d like to avoid running unnecessarily. For the purposes of this post you’ll have to suspend disbelief and pretend that negating a number is an expensive task. This example prints out a message so you can see when it actually gets called.

{{< highlight fsharp "style=tango" >}}
let Negate n =
    printfn "Negating '%A' this is hard work" n
    -n
 
val Negate : int -> int

> Negate 5;;
Negating '5' this is hard work
val it : int = -5
{{< /highlight >}}

Now, let’s use that function when writing another. This function takes two numbers, negates the first, then adds the second.

{{< highlight fsharp "style=tango" >}}
let addNegatedValue x y =
    let n = Negate x
    n + y
 
val addNegatedValue : int -> int -> int

> addNegatedValue 2 3;;
Negating '2' this is hard work
val it : int = 1
{{< /highlight >}}

If you call this function repeatedly the telltale message lets you know that the expensive negation function is called every time. We’d like to avoid that.

Perhaps currying can help. Let’s create a curried version of ‘addNegatedValue’ and see what happens.

{{< highlight fsharp "style=tango" >}}
let negateTwoThenAdd = addNegatedValue 2
 
val negateTwoThenAdd : (int -> int)
 
> negateTwoThenAdd 3;;
Negating '2' this is hard work
val it : int = 1
> negateTwoThenAdd 3;;
Negating '2' this is hard work
val it : int = 1
{{< /highlight >}}

This gives us a new function with the first argument (2 in this case) locked in, so we can call it passing only the 3 (or whatever other number we want to add to -2).

Unfortunately, as you can see above, when you use the curried function ‘negateTwoThenAdd’ a second time, it runs the negation code again. This makes sense. Currying locks in the argument 2 not the result of negating it. It doesn’t and shouldn’t get involved in optimizing out any lines of code in the function.

We need to explicitely ‘cache’ the negated value to avoid recalculating it.

{{< highlight fsharp "style=tango" >}}
let cachedAddNegatedValue x =
    let n = Negate x
    fun y -> n + y
 
val cachedAddNegatedValue : int -> (int -> int)
 
let cachedNegateTwoThenAdd = cachedAddNegatedValue 2
 
Negating '2' this is hard work
 
val cachedNegateTwoThenAdd : (int -> int)
{{< /highlight >}}

This time the Negation function fires when we create the curried version of the function. Of course it does, look at the code, it’s right there. Negate x then return a function.

We now have a handle ‘cachedNegateTwoThenAdd’ to that returned function. All that function does is accept another value and add ‘n’ (the result of negating ‘x’) to it.

{{< highlight fsharp "style=tango" >}}
> cachedNegateTwoThenAdd 3;;
val it : int = 1
{{< /highlight >}}

Bingo. We can call this all day and it will add any number we want to -2 without needing to rerun the ‘expensive’ negation operation.

I don’t want to gloss over the significance of this function being able to access ‘n’. It’s not passed as an argument, it’s just there, in scope, available. This is what’s known as a closure and it’s kind of a big deal as we’ll see shortly.

Before we take this code any further, let’s divert slightly to talk about closures and mutability. We’ve seen that ‘cachedNegateTwoThenAdd’ can access the variable ‘n’, however it can’t modify that variable. Even if you flag the variable as mutable. This code isn’t valid.

{{< highlight fsharp "style=tango" >}}
let addAndRememberTotal =
    let mutable n = 0
    fun x -> 
        n <- x + n
        n
{{< /highlight >}}

In fact if a variable is flagged as mutable you can’t use it at all in a closure, even if you only read it.

{{< highlight fsharp "style=tango" >}}
let addAndRememberTotal =
    let mutable n = 0
    fun x -> x + n
{{< /highlight >}}

Sorry for that little diversion, but mutability and closures are important for the rest of this post.

Where were we?

We have a solution that allows us to lock in a specific result of the expensive operation, we can then use that as many times as we want without needing to run the negate function again. That’s fine in so far as it goes, but it would be nice if we could cache the result for more than one input.

We should be able to throw all sorts of values at a function and have it cache return values so that subsequent calls for the same input don’t require another call to negate.

To do this, we can’t simply use a single cached value like an int. We need something that maps inputs to computed outputs. The snag is this cache needs to be mutable because every time we get a new input we want to compute an answer for it and store it in the cache. But, we just showed that closures can’t access mutable variables.

Except…they can…sort of. We can rope in some old mavericks, guys who don’t play by the rules, guys who eat mutability for breakfast.

Take a look at this little function for a clue to where this is heading.

{{< highlight fsharp "style=tango" >}}
let rememberTheEvens =
    let theList = List<int>()
    fun n ->
        if n%2 = 0 then theList.Add(n)
        theList 
{{< /highlight >}}

It’s a closure, but it’s calling the ‘Add’ method on a List. The List in this case is a .Net System.Collections.Generic List. We’re changing the state of the List by adding items to it, but from the perspective of F# it’s still the same list, the mutability is hidden behind a reference to the list, and that’s good enough for F# to get off our backs about it.

Once you realise you can do this it’s a short hop to the following code.

{{< highlight fsharp "style=tango" >}}
let memoizedNegation =
    let cache = Dictionary<int,int>()
    fun x ->
        if cache.ContainsKey(x) then
            cache.[x]
        else
            let res = Negate x
            cache.[x] <- res
            res
{{< /highlight >}}

This might look a little complicated but odds are you’ve written code just like this at some point. At it’s heart this is just a function that uses a dictionary to cache the return values keyed by the input values. For a given value the “expensive” function should only need to be called once. Subsequent calls for the same value can use the cached value.

The quirk here is that this is a closure, this function creates the cache then returns a different function that uses that cache. If you’ve followed everything I’ve written above this should be easy enough to grasp.

All this boilerplate code just to add caching to the ‘Negate’ function hardly seems worthwhile. There has to be a better way.

Of course there is. This is where the notion of functions as first class citizens really comes into it’s own. The following code is where this whole post has been heading all along, and it’s lifted directly from this post by Don Syme

Take a look at this code. Play a little spot the difference between this and the ‘memoizedNegation’ function above.

{{< highlight fsharp "style=tango" >}}
let memoize f =
    let cache = Dictionary<_, _>()
    fun x ->
        if cache.ContainsKey(x) then
            cache.[x]
        else
            let res = f x
            cache.[x] <- res
            res
{{< /highlight >}}

This function takes an argument f, the previous function specifically executed the ‘Negate’ function, this one executes ‘f’ whatever ‘f’ is. So, we can use this to add caching to any function. The other big difference is that we don’t limit the dictionary to ints, both the key and value are generics.

Other than those two differences, it’s basically the same function.

Here are two simple examples of the memoize function in action. Again, these are trivial examples that aren’t worth the effort, they just illustrate how to use the general ‘memoize’ function.

{{< highlight fsharp "style=tango" >}}
let increment n =
        printfn "Adding 1 to '%A'" n
        n + 1
 
let mIncrement =
    memoize (fun n -> increment n)
 
let add x y =
        printfn "Adding '%A' to '%A'" x y
        x + y
 
let mAdd =
    memoize (fun x y -> add x y)
{{< /highlight >}}

There’s one interesting problem with memoization and that is it’s behaviour on recursive functions. Take a look at the following naive recursive implementation of the factorial function. If we memoize it and try to get the factorial of 5 we see that the factorials of 4, 3, 2, 1 and 0 must be calculated. Trying to get the factorial of 5 again gives the result immediately because it’s cached. However the factorials of the lower numbers are not cached. If you try to get the factorial of 4 it must be calculated.

{{< highlight fsharp "style=tango" >}}
let rec fact x =        
    printfn "Getting factorial of '%A'" x
    if x < 1 then 1
    else x * fact (x - 1)
 
let mfact = memoize (fun n -> fact n)
 
> mfact 5;;
Getting factorial of '5'
Getting factorial of '4'
Getting factorial of '3'
Getting factorial of '2'
Getting factorial of '1'
Getting factorial of '0'
val it : int = 120
> mfact 5;;
val it : int = 120
{{< /highlight >}}

I’ll leave this here as a little exercise. Figuring out why the lower values are not cached is fairly trivial, figuring out what to do about it is a bit more of a challenge.