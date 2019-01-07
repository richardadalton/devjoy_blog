+++
date = "2017-02-06T17:43:00Z"
title = "Reading F#"
abstract="Even experienced C# developers can struggle to understand F# code. In this post I take a piece of F# code and break it down line by line to explain what it does."
tags = ["code", "c#", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png"
aliases = [
    "/2017/02/reading-f/",
]
+++

A tweet about some C# code rewritten in F# got me interested yesterday.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">&quot;An F# rewrite of a fully refactored C# Clean Code example&quot;. Amazing. <a href="https://t.co/mplbbH1knb">https://t.co/mplbbH1knb</a></p>&mdash; Jon Harrop (@jonharrop) <a href="https://twitter.com/jonharrop/status/824320112702394368?ref_src=twsrc%5Etfw">January 25, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

If you know a little F# it’s easy get sucked into thinking that having much fewer lines of code, and less noise generally makes F# code automatically better, cleaner, easier to read than C#. But, of course, that’s only true for people who know enough F# to read it.

When I see very smart C# devs unable to decipher what the F# code is doing, that gets me very interested.
 
<blockquote class="twitter-tweet" data-lang="en" data-conversation="none"><p lang="en" dir="ltr">got half way down and was lost but then again I don&#39;t know F# 😄</p>&mdash; Jonathan Channon (@jchannon) <a href="https://twitter.com/jchannon/status/824330128733966337?ref_src=twsrc%5Etfw">January 25, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
 
<blockquote class="twitter-tweet" data-lang="en" data-conversation="none"><p lang="en" dir="ltr">I know it&#39;s because I don&#39;t know F#, but that code is unreadable</p>&mdash; Cole Markham (@cole_markham) <a href="https://twitter.com/cole_markham/status/824636240997683201?ref_src=twsrc%5Etfw">January 26, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Pete Smith very kindly did his own [rewrite](https://gist.github.com/beyond-code-github/8711794c4d516cb6941d47274884b248) of the C# version.


### The F# Code
The point of this post is to explain the original F# code a little bit, for C# devs who are curious, but find it hard to follow. It’s no reflection on either C# developers or the F# language that there’s confusion. This is a new paradigm. There are concepts in F# that simply don’t exist in C#. There are also concepts that look like C#, but behave differently.

### The Types
First we’ll look at the type definitions.

{{< highlight fsharp "style=tango" >}}
module Discount =
    type Year = int
    type [<Measure>] percent
{{< /highlight >}}

Nothing exciting here. The code we’re writing is in a module called Discount. We’ll be able to import or ‘open’ that module when we want to use it later.

We create an alias (or Type Abreviation) of int called Year. This let’s us use ‘Year’ when defining other types, rather than the primitive int.

We also define percent as a unit of measure. F# can do amazing things with Units of Measure. This percentage example doesn’t necessarily show it off to full effect.

{{< highlight fsharp "style=tango" >}}
type Customer = Simple | Valuable | MostValuable
{{< /highlight >}}

Customer looks enough like an enum that most devs let it slide, but it’s more than that. Customer is a type with three possible values: Simple, Valuable, and MostValuable. These are not mere labels. This isn’t some layer of text over a numeric data type like an enum. They represent the full range of values for the Customer Type. In a sense, they are to Customer what ‘true’ and ‘false’ are to bool.

Let me repeat that, Customer can hold no other value than Simple, Valuable, or MostValuable. If you have a Customer, it can not hold an invalid value. it can not hold Null, or Nothing, or any out of range numeric value.

What we’re trying to do here is model the domain with types that make it impossible to capture impossible states.

AccountStatus is the first type that’s likely to completely throw a C# developer.

{{< highlight fsharp "style=tango" >}}
type AccountStatus = 
    | Registered of Customer * since:Year
    | UnRegistered
{{< /highlight >}}

This is actually similar to Customer, although the layout may suggest a difference that isn’t really there. An AccountStatus can be Registered or UnRegistered, just like a Customer can be Simple, Valuable, or MostValuable. For Registered accounts there’s extra information the Customer, and the Year they registered. For UnRegistered accounts there is no additional data, just the token UnRegistered.

This means that some valid values for the DataType AccountStatus include

{{< highlight fsharp "style=tango" >}}
Registered(MostValuable, 1)
Registered(Valuable, 6)
Registered(Simple, 1)
UnRegistered
{{< /highlight >}}

The following are invalid, they are syntax errors.

{{< highlight fsharp "style=tango" >}}
UnRegistered(MostValuable, 1)
Registered
Registered(1, Simple)
{{< /highlight >}}

Now, there’s a problem here. The Type definition makes Year look like a Year that an account has been active ‘since’. But later in the code it looks more like the number of years an account has been active ‘for’.

I’m not here to review or improve the code, just explain it. So, I’m mentioning that confusion here.

An Account Status can be UnRegistered. Or, it can be Registered with a Tuple of Customer and Year. Customer as we’ve seen can be Simple, Valuable or MostValuable. And Year is an integer.

That’s Algebraic Data Types. Define your types, compose them and use them. It’s no different to what you’ve always done in C#, however you haven’t had Sum Types, and you can’t define a tuple as simply as ‘Customer * Year’.

Some C# developers may like to think of AccountStatus as a BaseClass and Registered and UnRegistered as SubClasses. With Registered having additional fields. That is what’s going on under the hood, but I personally don’t give that a lot of thought.

By the way you can delete ‘since:’ in the definition of a Registered account, it serves only to tell you what the Year indicates. It doesn’t change the type in any way.

Those are the types we have to work with. Functional Programmers tend to lean quite heavily on types. I can definitely understand C# devs wondering whether ‘Year’ and ‘percent’ are really worth the effort in this case. Year in particular is troublesome because int isn’t necessarily the most robust way of representing a Year. If you’re going to introduce a new Type, maybe you should go all the way or don’t go at all. The confusion over what ‘Year’ actually means is troublesome.

That’s a debate for another day. But in this example at least the concept of Year is called out. The specific implementation, and underlying type may change later.

Let’s move on.

### The Functions

{{< highlight fsharp "style=tango" >}}
let customerDiscount = function
    | Simple    -> 1<percent>
    | Valuable  -> 3<percent>
    | MostValuable  -> 5<percent>
{{< /highlight >}}

customerDiscount is just a function that maps a Customer to an integer percent. How do I know? Well that’s the signature of the function.

{{< highlight fsharp "style=tango" >}}
Customer -> int 
{{< /highlight >}}

So, the valid inputs to this function are Simple, Valuable, and MostValuable. And the outputs you can see.

The way this function is written probably throws C# devs more than what it actually does. Let me rewrite it slightly.

{{< highlight fsharp "style=tango" >}}
let customerDiscount customer = 
    match customer with
    | Simple    -> 1<percent>
    | Valuable  -> 3<percent>
    | MostValuable  -> 5<percent>
{{< /highlight >}}

This is exactly the same function, it just gives the argument a name, and then pattern matches on it. Because this kind of function is so common, the alternative syntax is possible.

{{< highlight fsharp "style=tango" >}}
let yearsDiscount = function
    | years when years > 5  -> 5<percent>
    | years                 -> 1<percent> * years
{{< /highlight >}}

yearsDiscount is a function that maps an int to an int percent. That Year alias is getting more troubling. It seems to have vanished here in the code where it’s actually used. F# isn’t perfect, and it doesn’t write itself for you. Ambiguities can creep in.

Let’s stick to what this function is doing. The function is in the same simplified pattern matching syntax as customerDiscount. The first clause matches when the value passed to the function is greater than 5, and returns 5 . The second clause matches any other integer value, and calculates the result. The end result, 1% discount per year, capped at 5%.

Notice that the entire body of the functions are expressions. There’s no ‘return’ statement. A clause in the match maps to a value and that is the value if the function.

{{< highlight fsharp "style=tango" >}}
let accountDiscount = function
    | Registered(customer, years) -> customerDiscount customer, yearsDiscount years
    | UnRegistered                -> 0<percent>               , 0<percent>
{{< /highlight >}}

On, now things are getting interesting. The signature of accountDiscount is

{{< highlight fsharp "style=tango" >}}
AccountStatus -> int<percent> * int<percent>
{{< /highlight >}}

What does that mean?

We can pass in either a Registered account, with a Customer and number of years,
OR
We pass in unRegistered.

Those are the two possibilities for AccountStatus.

What do we get back?

{{< highlight fsharp "style=tango" >}}
int * int
{{< /highlight >}}

A tuple, containing two int percents.

The tuple contains the results of calling the customerDiscount function, and the yearsDiscount function.

Look again at the accountDiscount function. How does it know the types of the input, and the output values.

{{< highlight fsharp "style=tango" >}}
let accountDiscount = function
    | Registered(customer, years) -> customerDiscount customer, yearsDiscount years
    | UnRegistered                -> 0<percent>               , 0<percent>
{{< /highlight >}}

It pattern matches on Registered and UnRegistered, so the input must be an AccountStatus. Both match clauses evaluate to an int * int tuple. So, the function as a whole must always evaluate to that too.

If the input to the function is UnRegistered, then the result is 0, 0. So, no discount.
But look at the match on Registered. Remember the Registered AccountStatus has a payload of sorts. A Customer type and number of years in the form of a Customer * Year tuple.

In the match clause we destructure that tuple into two variables customer and years.

{{< highlight fsharp "style=tango" >}}
Registered(customer, years)
{{< /highlight >}}

And then pass those variables to the relevant discount function to produce an output tuple.

{{< highlight fsharp "style=tango" >}}
-> customerDiscount customer, yearsDiscount years
{{< /highlight >}}

There’s a lot going on there that isn’t familiar. It takes a little time to adjust. The on the fly translation that a C# dev needs to do in their head is quite a burden when you start reading (and even harder when you start writing) F#. But it does get easy very quickly, and after a while then inherent consistency starts to shine through.

{{< highlight fsharp "style=tango" >}}
let asPercent p = 
    decimal(p) / 100.0m
{{< /highlight >}}

Ok, let me have another little moan here. This function takes an int, in other words a percentage in this nice format: 5, and returns it as a decimal 0.05m.

So, asDecimal might be a better name. Having to convert back to a decimal likes this leads me to thing maybe a decimal might have done the job just as well. But I’m not here to judge, just explain.

{{< highlight fsharp "style=tango" >}}
let reducePriceBy discount price = 
    price - price * (asPercent discount)
{{< /highlight >}}

This looks pretty straightforward, surely there’s no functional voodoo going on here? Well, actually there is some very cool functional voodoo going on.

In C# land, the order of arguments for a function isn’t strictly speaking, important. Some developers come up with standards and best practices, but, basically as long as you’re consistent, it doesn’t really matter.

In languages like F# it matters a great deal.

On the face of it, the reducePriceBy function accepts 2 arguments, discount, and price.

Partial Application was one of the big things that popped my C# tinted eyes, when I first encountered F#. In simple terms it means that if you pass some of the arguments to a function, you get back a function that accepts the rest of the arguments.

So, if we pass a discount to reducePriceBy, we get back a function that accepts a price, and reduces it by that locked in discount.

That’s why discount is the first argument and price is the second. It’s hard to see a use for a function that accepts various discounts and applies them to some locked in price.

Through the wonder of partial application, and all the types and functions above, we get to the centrepiece of the program. If you had known F# from the start, your eye would have headed straight for this function to see what was going on.

{{< highlight fsharp "style=tango" >}}
let calculateDiscountedPrice account price = 
    let customerDiscount, yearsDiscount = accountDiscount account
    price
    |> reducePriceBy customerDiscount
    |> reducePriceBy yearsDiscount
{{< /highlight >}}

calculateDiscountedPrice takes an AccountStatus. Which is either Registered for a particular Customer, and number of years, or UnRegistered.

calculateDiscountedPrice also takes a price which is a decimal.

The accountDiscountFunction takes the AccountStatus and returns a tuple containing Customer Discount and Years Discount (both in the int format). These are stored in customerDiscount and yearsDiscount respectively. There’s that destructuring again.

So, what’s the logic of discounting a price? Here’s the important bit.

{{< highlight fsharp "style=tango" >}}
price
|> reducePriceBy customerDiscount
|> reducePriceBy yearsDiscount
{{< /highlight >}}

Let me rewrite that slightly.

{{< highlight fsharp "style=tango" >}}
price
|> (reducePriceBy customerDiscount)
|> (reducePriceBy yearsDiscount)
{{< /highlight >}}

Remember I said that passing a discount value to reducePriceBy would return a new function that accepts a price.
Well, that’s what we’re doing. The parenthesis above aren’t necessary but they show that partial application is going to produce two functions, one that reduces a price by the customer discount amount and a second that reduces a price by the years discount amount.

Or, to make a short story long…

{{< highlight fsharp "style=tango" >}}
let reducePriceByCustomerDiscount = reducePriceBy customerDiscount
let reducePriceByYearsDiscount = reducePriceBy yearsDiscount

price
|> reducePriceByCustomerDiscount
|> reducePriceByYearsDiscount
{{< /highlight >}}

The pipe forward operator |> simply takes the value on it’s left and passes it to the function on it’s right. You will occasionally hear people (like me a long time ago) say that the pipe forward operator passes a value on the left in as the ‘last argument’ to the function on the right.

As you can hopefully see that’s a bad way to think about it. The expression on the right of the pipe forward operator is evaluated and should produce a function. That expression might just be a function, or it might be a function that needs to be partially applied. It might in fact be any expression that evaluates to a function capable of accepting the value on the left of the operator.

And, when that value is piped into the function, the result of that can be piped on in the same way to the next function. As we see here.

The final little gimmick in this program is the test. There’s no test framework, or assert. Just a variable tests that will either be true or false.

{{< highlight fsharp "style=tango" >}}
let tests =
    [
        calculateDiscountedPrice (Registered(MostValuable, 1))  100.0m
        calculateDiscountedPrice (Registered(Valuable, 6))      100.0m
        calculateDiscountedPrice (Registered(Simple, 1))        100.0m
        calculateDiscountedPrice UnRegistered                   100.0m
    ] = [94.05000M; 92.15000M; 98.01000M; 100.0M]
{{< /highlight >}}

Here are the two lists.

{{< highlight fsharp "style=tango" >}}
[
    calculateDiscountedPrice (Registered(MostValuable, 1))  100.0m
    calculateDiscountedPrice (Registered(Valuable, 6))      100.0m
    calculateDiscountedPrice (Registered(Simple, 1))        100.0m
    calculateDiscountedPrice UnRegistered                   100.0m
]

[94.05000M; 92.15000M; 98.01000M; 100.0M]
{{< /highlight >}}

In all cases the price being used is 100.0m.

The AccountStatus values are as discussed earlier, either UnRegistered, or Registered along with a tuple of a Customer and an int.

Each of those calls to calculateDiscountedPrice will evaluate to a decimal, so we’ll end up with a list of decimals. If that list happens to match the list of decimals provided then ‘tests’ will be true, otherwise it will be false.

As it happens, it’s true

{{< highlight fsharp "style=tango" >}}
val tests : bool = true
{{< /highlight >}}