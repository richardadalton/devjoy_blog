+++
date = "2014-08-14T00:00:00Z"
title = "Multi Case (|A|B|)"
tags = ["functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png" 
+++

Playing Cards are a commonly used example of discriminated unions in F#. I’m not presuming that you already understand Discriminated Unions, but I’m also not going to explain them. You should be able to follow along and get a sense of how they work. If you’d like to read up on them try here.

A Rank is something that can have one of 13 values Ace through King. A Suit can have one of 4 values Hearts, Clubs, Diamonds or Spades. A Card can be represented as a Tuple of Rank and Suit.

{{< highlight fsharp "style=tango" >}}
type Rank = Ace|Two|Three|Four|Five|Six|Seven|Eight|Nine|Ten|Jack|Queen|King
type Suit = Hearts|Clubs|Diamonds|Spades
type Card = Rank * Suit
{{< / highlight >}}

As a brief aside take a look at the ‘*’ which indicates a tuple. Tuples are known as ‘product types’ their range of values are the product of the ranges of values of the types that make them up.

Let me try that again in English. 13 Ranks * 4 Suits = 52 Cards. I’ll pretend Jokers don’t exist, this series is long enough already.

{{< highlight fsharp "style=tango" >}}
(Two, Clubs)
(Queen, Hearts)
(Ace, Spades)
{{< / highlight >}}

Cards have an interesting characteristic, every single one of them is either Red or Black. So, here’s a little challenge. Using what you’ve learned so far, write a function takes a card and returns a message like the following

{{< highlight fsharp "style=tango" >}}
(Two, Clubs) -> "The Two of Clubs is Black"
(Queen, Hearts) -> "The Queen of Hearts is Red"
(Ace, Spades) -> "The Ace of Spades is Black"
{{< / highlight >}}

If you’re having trouble with that the following easier version will get you a passing grade.

{{< highlight fsharp "style=tango" >}}
(Two, Clubs) -> "Black"
(Queen, Hearts) -> "Red"
(Ace, Spades) -> "Black"
{{< / highlight >}}

There are lots of ways we can approach this using the Single Active Patterns (both the Total and Partial varieties). All have drawbacks of one kind or another.

### Single Total

Here’s how we could do it with a Single Total Active Pattern

{{< highlight fsharp "style=tango" >}}
let (|Red|) (card: Card) =
    match card with
    | (_, Hearts) | (_, Diamonds) -> true
    | _ -> false

let DescribeCard (card: Card) =
    match card with
    | Red true -> sprintf "Red"
    | Red false -> sprintf "Black"
{{< / highlight >}}

That’ll get us the passing grade but it’s pretty lousy. Black has vanished as a concept, all we have is “Not Red”, and within our match expression we’ve lost the details of the card, all we have is the boolean that tells us if it’s Red or not.

### Partials
We can do better with two partials.

{{< highlight fsharp "style=tango" >}}
let (|Black|_|) (card: Card) =
    match card with
    | (_, Clubs) | (_, Spades) -> Some card
    | _ -> None

let (|Red|_|) (card: Card) =
    match card with
    | (_, Hearts) | (_, Diamonds) -> Some card
    | _ -> None

let DescribeCard (card: Card) =
    match card with
    | Black (rank, suit) -> sprintf "The %A of %A is Black" rank suit 
    | Red (rank, suit) -> sprintf "The %A of %A is Red" rank suit 
{{< / highlight >}}

That’s better, but if you try that example you’ll see F# throws a warning in the DescribeCard function. It claims that we have an incomplete match pattern. F# is concerned that a card might come along that doesn’t match either Black or Red.

We know we’ve got our bases covered, but the use of Partials hides that fact from F#. What happens if a card comes along that gets ‘None’ from both (|Black|) and (|Red|). I knew those damn Jokers would bite us in the ass.

We could throw in a dummy catch all case that would get rid of the warning, but where’s the fun in that?

This, as it happens is an ideal scenario for a Multi-Case Active Pattern. We know that the full range of possible values for a card (i.e. all 52 cards) must map to one of a small set of possibilities (Red and Black). We can make that fact explicit using a Multi-Case active pattern and kill off a compiler warning in the process.

### Multiple

{{< highlight fsharp "style=tango" >}}
let (|Red|Black|) (card: Card) =
    match card with
    | _,Hearts|_,Diamonds -> Red card
    | _ -> Black card

let DescribeCard (card: Card) =
    match card with
    | Red (rank, suit) -> sprintf "The %A of %A is Red" rank suit
    | Black (rank, suit) -> sprintf "The %A of %A is Black" rank suit
{{< / highlight >}}

The partials are gone. There’s no more ‘Some card’ or ‘None’. Every card is classified as either a ‘Black card’ or a ‘Red card’. Our match expression doesn’t throw a warning any more because F# can understand that Red and Black are the Only possibilities for cards, and we handle them both.

### Bringing It All Together
Let’s finish with a nice meaty example that brings together the various types of pattern we’ve covered. We’ll use Single and Multiple Active Patterns to pattern match and describe a hand of Blackjack cards. To keep things simple Ace will count as 11 only, it won’t also count as 1.

The points attached to each card are determined by the Rank. We’ll start with a function that scores a card.

{{< highlight fsharp "style=tango" >}}
let CardScore (card: Card ) = 
    match fst card with
    | Two -> 2
    | Three -> 3
    | Four -> 4
    | Five -> 5
    | Six -> 6
    | Seven -> 7
    | Eight -> 8
    | Nine -> 9
    | Ten | Jack | Queen | King-> 10
    | Ace -> 11
{{< / highlight >}}

Our Active Patterns are going to look at hands and tell us something about them. Every hand has a Score so (|Score|) can be a Single Total Active Pattern. And, it’s really easy to write.

{{< highlight fsharp "style=tango" >}}
let (|Score|) (hand: Card List) =
    hand
    |> List.sumBy CardScore
{{< / highlight >}}

The number of cards is also a Single Total Active Pattern, and a useful piece of information, as we’ll see in a moment.

{{< highlight fsharp "style=tango" >}}
let (|Cards|) (hand: Card List) =
    List.length hand
{{< / highlight >}}

We now have everything we need to fit a hand of cards into one of five possibilities. A Multiple Active Pattern will work. Notice how the code is about as close to a description of the rules as you can get. We literally list the various types of hands. Blackjack is when we have 2 cards with a combined score of 21, A Pair is when we have two cards with the same Rank and we don’t care about the Suits.

{{< highlight fsharp "style=tango" >}}
let (|Bust|Blackjack|TwentyOne|Pair|Under|) (hand: Card List) =
    match hand with
    | Score s when s > 21 -> Bust
    | Cards 2 & Score 21 -> Blackjack
    | Score 21 -> TwentyOne
    | [(r1,_); (r2,_)] when r1 = r2 -> Pair r1
    | Score s -> Under s
{{< / highlight >}}

Some of the cases simply identify the type of hand Bust, Blackjack, TwentyOne. Others package some information with the hand type. Pair tells us the Rank of the pair. Under tells us the score. This Multi-Case active pattern is working with Single Case Active Patterns, and together they make the following function really easy to write.

{{< highlight fsharp "style=tango" >}}
let DescribeHand (hand: Card List) = 
    match hand with
    | Bust -> "You're over 21, you're bust"
    | Blackjack -> "BLACKJACK!!! Yeah!!!"
    | TwentyOne -> "TwentyOne, you should probably stick"
    | Pair r -> sprintf "A pair of %As You can split" r
    | Score s -> sprintf "You have %d, want me to hit you?" s
{{< / highlight >}}

And here’s where it gets really fun, if for any reason someone goes and adds a new value to the Active Pattern, a new type of hand, then our DescribeHand function will fail to compile. We would no longer be covering all of the possibilities.

You don’t get that kind of protection when you use partials and catch-alls.

### Caveats
There are one or two things to note about Multi-Case active pattern.

They can have a maximum of 7 values. Our example above had only 5, but there are 10 named Poker Hands, so for Poker we’d need take a slightly different approach. 

Typically if you are looking at a lot of values putting them in separate Single Partial Active Patterns may make more sense, despite the concerns expressed above.

They can not be Partial. Multi-Case Patterns exist precisely to give the exhaustive pattern matching. If you are going to require catch all clauses then you might as well use Single Partial Patterns.

They can not have additional arguments beyond the one argument that they match on. You can define a multi case active pattern function with additional arguments but you will receive an error if you attempt to use it. That may seem odd, but as we’ll see in the next (and final) post, you can partially apply a Multi-Case pattern to create one that accepts one argument, and that can then be used.

We’ve now covered all of the different types of Active Patterns, we’ve also touched on a number of other areas within F# from Option Types to Partial Application, Tuples to Discriminated unions. If you’ve stayed with me then you should feel like you “get” active patterns, and you may have picked up some extra F# along the way.

Active Patterns are incredibly flexible, we've only scratched the surface of all the way’s they can be used. 

There is [one more post]({{< ref "choices-and-nesting.md" >}}) to go, it will cover some strange and wonderful things you can do with what you’ve learned. We’ll partially apply the Multi-Case patterns, and we’ll nest active patterns to match complex hierarchical data structures.