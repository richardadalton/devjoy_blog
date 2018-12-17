+++
date = "2014-06-02T00:00:00Z"
title = "Recursion"
tags = ["functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png"
series = ["Thinking Functionally"]
series_weight = 05
+++

This post looks at a hugely important part of functional programming, Recursion. In simple terms this means writing a function that calls itself.

There are endless examples of using recursion to figure out Fibonacci numbers, or process lists. This post will be a little more complicated but hopefully is simple enough that you’ll be able to follow along.

We’re going to teach F# to play the perfect game of Tic-Tac-Toe. The game is a favourite for kids, but it quickly becomes boring for adults. The reason for this is that it is relatively easy to look ahead a few moves and play a perfect game. Writing a program that can look ahead and play perfectly is a nice little programming challenge, so let’s get to it.

### Modelling The Tic-Tac-Toe Board
The first question we need to address is how to represent the board. A nine item array? Perhaps with 1, -1 and 0 for X, O and Blank respectively. That works.

Or we could just avoid modelling the board altogether. Instead of updating a “Board” to mark the squares occupied by the players, we could just keep two lists of the claimed squares.

If we imagine a board layout like this

{{< highlight text "style=tango" >}}
1 2 3
4 5 6
7 8 9
{{< /highlight >}}

Then the following game position

{{< highlight text "style=tango" >}}
X X O
. O .
X . .
{{< /highlight >}}

can be represented as

{{< highlight text "style=tango" >}}
X's Squares: [1;2;7]  
O's Squares: [5;3]
{{< /highlight >}}

### Some Helper Functions

Once we know how a positions are represented we can write functions to reason about them. From a given position we can find the list of squares that remain available.

{{< highlight fsharp "style=tango" >}}
let Available (player: int list) (opponent: int list) =
    [1..9]
    |> ExceptList (List.append player opponent)  
{{< /highlight >}}

We combine the player and opponent lists into one list, then exclude that from the list [1..9]. What remains are the available squares. Simple. ExceptList is just a helper function that wraps a List.filter.

{{< highlight fsharp "style=tango" >}}
let ExceptList list = List.filter (fun n -> not (list |> Contains n))
{{< /highlight >}}

That Contains function is another helper function, It’s just a cleaner way of doing a List.exists.

{{< highlight fsharp "style=tango" >}}
let Contains number = List.exists (fun n -> n = number)
{{< /highlight >}}

### Seeing A Win
The next question is how to we know if a player has won. Each player’s squares are in their own list, so all we have to do is check their claimed squares and see if any three are connected.

Yet again there are various ways of doing this, here’s a really simple way. There are only 8 sets of winning squares. The three rows, three columns and the two diagonals. If a players list of squares contains any of these 8 patterns then it’s a win.

{{< highlight fsharp "style=tango" >}}
let wins = [[1;2;3];
            [4;5;6];
            [7;8;9];
            [1;4;7];
            [2;5;8];
            [3;6;9];
            [1;5;9];
            [3;5;7]]
{{< /highlight >}}

The IsWin function looks at each of the 8 win patterns and checks if any of them is contained in the players squares.

{{< highlight fsharp "style=tango" >}}
let IsWin (squares: int list) = 
    wins |> List.exists (fun w -> ContainsList squares w)
{{< /highlight >}}

It uses another helper function ContainsList that checks if a list contains all of the elements of another list.

{{< highlight fsharp "style=tango" >}}
let ContainsList list = List.forall (fun n -> list |> Contains n)
{{< /highlight >}}

A Draw is even easier to identify. If there are no squares left then it’s a draw.

{{< highlight fsharp "style=tango" >}}
let IsDraw player opponent = List.isEmpty (Available player opponent)
{{< /highlight >}}

### What’s a good move?
So, we know how to tell if the board shows a win or a draw, but how can we make our program look at a given position and decide on a good move. Let’s start by imagining that we have a function that can assign a score to a position. If such a function existed we could just get a list of available moves and pick the one with the highest score. Like This.

{{< highlight fsharp "style=tango" >}}
let BestMove (player: int list) (opponent: int list) =
    Available player opponent
    |> List.maxBy (fun m -> Score (m::player) opponent) 
{{< /highlight >}}

List.maxBy applies the given function to each item in the list, and selects the item that gives the largest result. Remember the list contains available squares. The function combines each available move with the players current position and gets the score for each.

Now, what might the Score function look like?

{{< highlight fsharp "style=tango" >}}
let Score (player: int list) (opponent: int list) =
    if (IsWin player) then 1
    else if (IsDraw player opponent) then 0
    else ???
{{< /highlight >}}

If the position is a win for the player we consider it to have a score of 1. For a draw we assign a score of 0. But, what if the game isn’t over yet? How can we score that position?

This is where we get into the realm of recursion. If we assume that both players will play their best possible moves from this position forward, we can predict how the game will end, that end position can then be used to score the current position. Let’s see that in action.

{{< highlight fsharp "style=tango" >}}
let rec Score (player: int list) (opponent: int list) =
    if (IsWin player) then 1
    else if (IsDraw player opponent) then 0
    else
        let opponentsBestMove = BestMove opponent player
        let opponentsNewPosition = opponentsBestMove::opponent
        -Score opponentsNewPosition player
{{< /highlight >}}

There are a few things going on here so let’s take them in turn. We need to figure out the best move for the opponent. We already have a function for that, the BestMove function that we started with. If we flip the opponent and player lists of squares around, we can use BestMove to find the best move from the opponents perspective.

Knowing the best move for the opponent we can add it to their current position. Now, we have a new position to score. It may be that the opponents move won them the game, or gives a draw. Or it may be that the game is still not finished. We have a function that can score those various options, in fact it’s the function we’re writing (Score), so we simply call it, passing it the opponents potential new position.

Note that if the Score function finds a win for our opponent it will return a score of 1. We negate that because a win for our opponent is a loss for us.

The key insight here is that while scoring the opponents position, the Score function will go back and forth between moves for the player and moves for the opponent. In parallel to the moves going back and forth, that negate operation means the score will go back and forth between 1 and -1.

Notice that we’ve added ‘rec’ to the Score function to indicate that it will be called recursively.

### Mutual Recursion
Notice also that the Score function calls the BestMove function, which is the function we started with and which in turn calls Score. This is mutual recursion, and it poses a particular problem in F#. A called function must defined be above the call to it. How can we do that when we have two functions that call each other?

Well, like this…

{{< highlight fsharp "style=tango" >}}
let rec Score (player: int list) (opponent: int list) =
    if (IsWin player) then 1
    else if (IsDraw player opponent) then 0
    else
        let opponentsBestMove = BestMove opponent player
        let opponentsNewPosition = opponentsBestMove::opponent
        -Score opponentsNewPosition player

and BestMove (player: int list) (opponent: int list) =
    Available player opponent
    |> List.maxBy (fun m -> Score (m::player) opponent) 
{{< /highlight >}}

The ‘and’ keyword allows us to define mutually recursive function. So, here’s the finished code.

{{< highlight fsharp "style=tango" >}}
let wins = [[1;2;3];
            [4;5;6];
            [7;8;9];
            [1;4;7];
            [2;5;8];
            [3;6;9];
            [1;5;9];
            [3;5;7]]

let Contains number = List.exists (fun n -> n = number)

let ContainsList list = List.forall (fun n -> list |> Contains n)

let ExceptList list = List.filter (fun n -> not (list |> Contains n))

let Available (player: int list) (opponent: int list) =
    [1..9]
    |> ExceptList (List.append player opponent)  

let IsWin (squares: int list) = 
    wins |> List.exists (fun w -> ContainsList squares w)
 
let IsDraw player opponent =
    List.isEmpty (Available player opponent)

let rec Score (player: int list) (opponent: int list) =
    if (IsWin player) then 1
    else if (IsDraw player opponent) then 0
    else
        let opponentsBestMove = BestMove opponent player
        let opponentsNewPosition = opponentsBestMove::opponent
        -Score opponentsNewPosition player

and BestMove (player: int list) (opponent: int list) =
    Available player opponent
    |> List.maxBy (fun m -> Score (m::player) opponent) 
{{< /highlight >}}

There you have it, 37 lines of code to create the perfect Tic-Tac-Toe brain. To run the code simply call the BestMove function, passing it the two lists representing the player’s squares. For the first move those lists will be empty.

{{< highlight fsharp "style=tango" >}}
> BestMove [] [];;
val it : int = 1
{{< /highlight >}}

Remember that the first list passed to BestMove is the squares occupied by the player that is about to move, the second are the opponent’s squares. So, to find the best response to the move above, you would do the following.

{{< highlight fsharp "style=tango" >}}
> BestMove [] [1];;
val it : int = 5
{{< /highlight >}}

The first move may take up to 30 seconds to find a move, it has to search the complete game tree. Subsequent moves will be significantly faster as the options get narrowed down.