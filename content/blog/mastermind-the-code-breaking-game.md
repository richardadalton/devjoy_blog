+++
date = "2015-01-07T17:43:00Z"
title = "Mastermind: The Code Breaking Game"
abstract="Mastermind is a classic game of permutations and code breaking. It also makes for a really interesting coding challenge."
tags = ["code", "f#"]
categories = ["how to"]
banner = "img/banners/mastermind.png"
+++

![Mastermind](/img/mastermind.png)

Mastermind, is a code breaking game for two players.

A “Code Maker” creates a secret sequence of colour pegs. A “Code Breaker” must break the code by taking guesses and working with the feedback from the Code Maker. Feedback is given using Black and White Pegs.

A correct colour in the correct position is acknowledged with a Black Peg
A correct colour in the wrong position is acknowledged with a White Peg
The position of these pegs is not significant. So, a black peg indicates that one of the guessed pegs is correct, but not which one.
See here for a detailed description of the game.

For a programmer both the Code Maker and Code Breaker roles are interesting challenges.

The Code Maker role most compare a guess and a secret code and generate the correct feedback.

The Code Breaker role must issue guesses and figure out the secret code based on the feedback.

Before we look at either role, lets create some types that we can work with.

{{< highlight fsharp "style=tango" >}}
type Colour = Red|Orange|Yellow|Green|Blue
type Answer = {Black: int; White: int}
{{< /highlight >}}

The Colours are a discriminated union of the 5 colours that can be used in the code. We also have an Answer that contains a certain number of Black and White pegs.

Each code consists of 4 pegs and colours can be duplicated. We don’t enforce that with types, in fact it’s quite easy to make the code work for arbitrary code lengths. There is one function in the current code that locks us into a code length of 4. I’ll discuss that towards the end of this post.

## The Code Maker
Let’s start with a function that can compare a secret code with a guess and return the black/white pegs answer. I’ve put a worked example in the comments so you can follow what’s happening.

{{< highlight fsharp "style=tango" >}}
// E.g.
// code     = Red, Orange, Yellow, Blue
// guess    = Yellow, Yellow, Green, Blue
// expected = {Black = 1 (Blue); White = 1 (Yellow)}
let check code guess =
    let IsMatch t = fst t = snd t

    // right = [(Blue, Blue)]
    // wrong = [(Red, Yellow); (Orange, Yellow); (Yellow, Green)]
    let right, wrong =
        List.zip code guess
        |> List.partition IsMatch
 
    // Number of Black Pegs
    // 1 (Blue, Blue)
    let rightColourRightPosition =
        List.length right

    // Number of White Pegs
    // wrongCode  = [Red; Orange; Yellow]
    // wrongGuess = [Yellow; Yellow; Green]
    let wrongCode, wrongGuess = List.unzip wrong 

    // E.g. when colour = Yellow, result = 2
    let howManyOfThisColourOutOfPlace colour =
        wrongGuess
        |> List.filter(fun c -> c = colour)
        |> List.length

    // Number of White Pegs
    // 1 (Yellow) Although Yellow is guessed twice, there is only one Yellow in the code, so result is 1
    let rightColourWrongPosition =
        wrongCode                                                                          // [Red; Orange; Yellow]
        |> Seq.countBy(id)                                                                 // seq [(Red, 1); (Orange, 1); (Yellow, 1)]
        |> Seq.map (fun group -> (snd group, howManyOfThisColourOutOfPlace (fst group)))   // seq [(1, 1); (1, 0); (1, 2)] (fst is occurences in code, snd is occurences in guess)
        |> Seq.sumBy Math.Min                                                              // For each colour, sum the lesser of occurences in code and in guess

    {Black = rightColourRightPosition; White = rightColourWrongPosition}
{{< /highlight >}}

The Code Maker has visibility of the Secret Code (because it create it) and the guess (because the Code Breaker asks to have a guess checked). This will be more interesting when we look at the signature for the solve function.

![Mastermind Secret](/img/mastermind-secret.png)

Let’s not get ahead of ourselves. How to we check a guess against a secret code and calculate the number of Black and White pegs?

We zip the secret code and the guess (which are both lists of colours) this gives us a list of tuples, where each tuple represents the colour of the two pegs at the same position in the code and the guess.

We then partition that list into tuples where the fst and snd match (same colour, same position) and where fst and snd don’t match.

The number of Black pegs is simply the length of the list of tuples that matched.

The number of white pegs is calculated by taking the colours from the list of non-matching tuples and figuring out how many of the colours in the code are also in the guess. Since we partitioned the list, none of the pegs that were in the right place will get in the way of this calculation.

Follow along with the comments in the code and it should be clear.

## The Code Breaker
On the face of it writing the code for the Code Breaker seems like it would be harder than for the Code Maker. It has to come up with guesses and then correlate the feedback from all the guesses to crack the code. We also can’t pass our secret code to the Code Breaker because that’s not information that it should have.

What should we pass to the solve function? It should be able to come up with guesses by itself. The only thing it needs is a way of asking the Code Maker to check a guess against the code. So, it needs a way of calling the check function. But it can’t pass the secret code to the check function, it can only pass the guess.

## Enter Partial Application.

We can wrap a secret code up in a closure by partially applying the check function. This gives us a function that just accepts a guess and returns an answer. From the Code Breaker’s point of view that’s exactly how the Code Maker should work.

let secretCodeChecker = check [Red; Orange; Yellow; Blue]
Armed with that function the logic of the code breaker is pretty simple.

{{< highlight fsharp "style=tango" >}}
let solve checkFunction =
    let filterPossibilities possibilities guess =
        let answer = checkFunction guess
        possibilities
        |> List.filter (fun potential -> (check guess potential) = answer)

    let rec solve_iter possible =
        match possible with
        | head::[] -> head
        | head::_ -> solve_iter (filterPossibilities possible head)
        | _ -> raise CanNotBeSolved

    solve_iter possibleCodes
{{< /highlight >}}

Start with all possibleCodes, take the first of them and guess it. Use the response from the Code Maker to filter out codes that are no longer possible. Repeat until there’s only one possibility left.

There’s a quick and easy way to generate all possible codes

{{< highlight fsharp "style=tango" >}}
let possibleCodes =
    seq {
            for i in ColourList do
                for j in ColourList do
                    for k in ColourList do
                        for l in ColourList do
                            yield [i;j;k;l]
    }
    |> List.ofSeq
{{< /highlight >}}

The problem with this is that it locks us into codes of length 4 even though nothing else in our solution has that restriction. A more general recursive function would be nice here, but let’s not waste time on that right now. Let’s get our Code Breaker working.

The most intersting part of breaking the code is filtering out potential codes based on feedback from a guess.

This ridiculously simple to do. We supply a guess to the Code Maker and get back an answer. We then take all of the remaining possible secret codes and we check our guess against each (using the non-partially applied check function) and we keep only the codes that result in the same answer that the Code Maker gave us.

To figure out the secret code that’s wrapped up in the secretCodeChecker function, we just pass it to solve

{{< highlight fsharp "style=tango" >}}
> solve secretCodeChecker;;
val it : Colour list = [Red; Orange; Yellow; Yellow]
{{< /highlight >}}

And what about those nested loops that produce all possible codes. How can be turn that into a more general recursive function?

I’ll leave that for you as an exercise. If you look up the [‘Making Change’ example in Structure and Interpretation of Computer Programs](https://mitpress.mit.edu/sicp/full-text/book/book-Z-H-11.html#%_sec_1.2.2) here you’ll be in the right area, however instead of simply counting the number of possibilities, you need to actually capture and return them.

#### Recommended Reading
[The Structure and Interpretation of Computer Programs](https://mitpress.mit.edu/sicp/full-text/book/book.html)