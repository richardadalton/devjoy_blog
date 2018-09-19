+++
date = "2014-07-01T00:00:00Z"
title = "Single Total (|A|)"
tags = ["functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png"
+++

[Part 1]({{< ref "pattern-matching.md" >}}) of this series was mainly sharpening the axe by covering some basics like Pattern matching. I also gave a general sense of what active patterns are (functions that can be used when pattern matching, such as in match expressions). Now it’s time to dig into the details.

As I mentioned previously there are arguably 5 variations of active patterns. This post will cover the first of those, the Single Total Active Pattern.

### Matching Against Abstractions
When we looked at plain old pattern matching we extracted and matched against values that were already there, by which I mean matching against a tuple like (21, 8, 2014) allowed us to match on the values 21, 8 and 2014 or any combination of them, but we couldn’t match against a values like ‘August’ or ‘Leap Year’.

Active Patterns allow us to do just that, we can take an input, transform it in some way and then match against the result of that transformation.

Let’s try a simple example.

{{< highlight fsharp "style=tango,hl_lines=1">}}
let (|UpperCaseCount|) (str: string) =
    str.ToCharArray()
    |> Array.filter (fun c -> c.ToString() = c.ToString().ToUpper())
    |> Array.length

let UseThePatternLuke (str: string) =
    match str with
    | UpperCaseCount 4 -> "Bingo 4 is the magic number"
    | UpperCaseCount u -> sprintf "Nah %d upper case characters is no good" u
{{< / highlight >}}

### Active Recognizers
The first thing to note is the highlighted line. In particular those funny parenthesis around (|UpperCaseCount|). Those (|…|) are “banana clips” and they denote a special kind of function known as an ‘Active Recognizer’. These functions do the heavy lifting for Active Patterns. They accept the source data, break it up, transform it and output it in a form that can be matched against.

From the perspective of match expressions and assignments the pattern matching works exactly like the plain old pattern matching we saw in the last post. The difference with Active Patterns is that the Active Recognizer function has gotten in and transformed the data before the matching happens.

The ”banana clips” above only enclose one value so this is a Single Total Active Pattern. The significance of this will become more apparent when we look at the remaining kinds in subsequent posts.

We’re matching against a string, but the property of the string we’re interested in is the number of upper case characters. So, we define an active recognizer that takes a string, and returns the number of upper case characters.

Apart from those Banana Clips it looks like an ordinary function. The Single Total Active Pattern can be a little hard to explain because simply using a function almost always seems like a better idea. If you’re skeptical, stay with me (I’ve been there).

For simple pattern matching, there’s just the “match x with” code, or the destructuring assignment. For Active Patterns you define the active recognizer separate from the pattern match. Basically you pull some logic out into it’s own function. Nothing magical.

Here’s the same code with some scribbles to try and convey the relationship between the active recognizer and the pattern matching.

{{< figure src="https://res.cloudinary.com/devjoy/image/upload/v1537270726/devjoy_blog/SingleTotalActivePattern.jpg" title="Single Total Active Pattern" >}}

And here is an example which is simple enough for anyone to understand, but where just using functions might not have been as clean.

{{< highlight fsharp "style=tango,hl_lines=27-28" >}}
let (|IsPalindrome|) (str: string) =
    str = String(str.ToCharArray() |> Array.rev)

let (|UpperCaseCount|) (str: string) =
    str.ToCharArray()
    |> Array.filter (fun c -> c.ToString() = c.ToString().ToUpper())
    |> Array.length

let (|LowerCaseCount|) (str: string) =
    str.ToCharArray()
    |> Array.filter (fun c -> c.ToString() = c.ToString().ToLower())
    |> Array.length

let (|SpecialCharacterCount|) (str: string) =
    let specialCharacters = "!£$%^"
    str.ToCharArray()
    |> Array.filter (fun c -> specialCharacters.Contains(c.ToString()))
    |> Array.length


let (|IsValid|) (str: string) =
    match str with
    | UpperCaseCount 0 -> (false, "Must have at least 1 upper case character")
    | LowerCaseCount 0 -> (false, "Must have at least 1 lower case character")
    | SpecialCharacterCount 0 -> (false, "Must have at least 1 of !£$%^")
    | IsPalindrome true -> (false, "A palindrome for a password? What are you thinking?")
    | UpperCaseCount u & LowerCaseCount l & SpecialCharacterCount s -> 
        (true, sprintf "Not a Palindrome, %d upper, %d lower, %d special." u l s)
{{< / highlight >}}

I’ve actually defined a couple of different Active Recognizers, each of which transform the string in different ways. The pattern match can then use any combination of the four patterns.

We can match against literal values like true and 0, or we can match against variables as in the last case.

The highlighted line shows one of the real advantages of active patterns over simple functions. We call three functions, store the returned values and use them, all in one line.

I was a little cheeky, even my IsValid “function” is actually an Active Recognizer, I can use it as follows

{{< highlight fsharp "style=tango" >}}
let checkPassword (password: string) =
    match password with
    | IsValid (true, _) -> "OK"
    | IsValid (false, reason) -> reason
{{< / highlight >}}

This would also work

{{< highlight fsharp "style=tango" >}}
let checkPassword (password: string) =
    match password with
    | IsValid (false, reason) -> reason
    | _ -> "OK"
{{< / highlight >}}

One final quirk of the Single Total Active Pattern is that you can use it like this.

{{< highlight fsharp "style=tango" >}}
> let (IsValid result) = "$TArAT$";;

val result : bool * string =
  (false, "A palindrome for a password? What are you thinking?")
{{< / highlight >}}

The value on the right of what looks like an assignment is sent to the Active Recognizer, and the result of the Active Recognizer is then bound to the variable ‘result’.

What’s going on here is simply the same Destructuring Assignment we saw in the first post, but using an Active Pattern instead of simple Pattern Matching. For more on this, and on the Single Total Active Pattern I strongly recommend Luke Sandell’s excellent (and concise) blog post.

Don’t get too hung up the Single Total Active Pattern. In many cases a simple function will work and be as clear or maybe even clearer than the Active Pattern equivalent.

That said, understanding what’s going on with this type of Active Pattern will make it very easy to grasp the rest, and once you know how to use a new tool, it becomes easier to see places where it can work.