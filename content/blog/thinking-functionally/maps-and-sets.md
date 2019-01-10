+++
date = "2014-07-21T17:43:00Z"
title = "Maps and Sets"
tags = ["functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png"
series = ["Thinking Functionally"]
series_weight = 06
aliases = [
    "/2014/07/learning-to-think-functionally-types-maps-and-sets/",
]
+++

In the most recent post in this series I implemented Tic-Tac-Toe using recursion to find the best moves. The point of that post was the recursion and I took the simplest approach I could think of to represent the actual board and moves.

I used two lists of ints, one for each player’s list of occupied squares. The board itself wasn’t explicitly represented at all, it could be inferred from the two lists.

One problem with this is that there are only 9 possible squares, but our lists could hold any integers, or could both hold the same integer. It was not only possible but quite easy to represent an invalid state.

For this post I’ll turn to this issue and try to use the type system to give us a more sensible model of the game.

{{< highlight fsharp "style=tango" >}}
type Player = X | O

type Position = TopLeft | TopMiddle | TopRight 
                | MiddleLeft | Center | MiddleRight 
                | BottomLeft | BottomMiddle| BottomRight

type Board = Map<Position, Player Option>
{{< /highlight >}}

I’ve represented a Player as being X or O, and the Position type has nine possible values, i.e. the nine squares on the board.

We move away from the idea in the last post of just storing each players squares and instead we use a Map from Position to a Player Option. This means that we can look at any square on the board and see if it is owned by X, O or by neither.

This fairly simple change has immediately solved the two invalid state issues I described above. A given square can not belong to both players simultaneously, and player can only own squares on the board, we can’t just “invent” new positions as we could by putting extra integers into the lists in the earlier example.

Let’s initialize an empty board so that we can start playing.

{{< highlight fsharp "style=tango" >}}
let NewBoard: Board = 
        Map [ (TopLeft, None); (TopMiddle, None); (TopRight, None); 
              (MiddleLeft, None); (Center, None); (MiddleRight, None); 
              (BottomLeft, None); (BottomMiddle, None); (BottomRight, None) ]
{{< /highlight >}}

I couldn’t figure out how to do a nice “For All” Positions, Map to None, other than by using Reflection which just seemed cumbersome for such a simple task.

Wins are defined in the same way as in the last post, but it’s a little more readable now because each position is named.

{{< highlight fsharp "style=tango" >}}
let wins = set [ set [ TopLeft; TopMiddle; TopRight ] ;
                 set [ MiddleLeft; Center; MiddleRight ] ;
                 set [ BottomLeft; BottomMiddle; BottomRight ] ;
                 set [ TopLeft; MiddleLeft; BottomLeft ] ;
                 set [ TopMiddle; Center;  BottomMiddle ] ;
                 set [ TopRight; MiddleRight; BottomRight ] ;
                 set [ TopLeft; Center; BottomRight ] ;
                 set [ TopRight; Center; BottomLeft ] ; ]
{{< /highlight >}}

Now we need a little helper function that will allow us to find all the squares with particular contents, i.e. All Empty squares, all squares for X or all squares for O.

{{< highlight fsharp "style=tango" >}}
let FindPositions (player: Player Option) (board: Board) =
    board
    |> Map.filter (fun _ mark -> mark = player) 
    |> Map.toSeq
    |> Seq.map fst
    |> Set.ofSeq
{{< /highlight >}}

This looks a little involved, but it’s very simple. We filter the board (which you’ll recall is a Map from Position to Player Option). We look for all squares with a mark that matches the Player we are looking for. This will work whether we pass in X, O or None.

The filter returns another Map, we want to split the Keys (Positions) from the Values (X, O, None). We convert the Map to a Sequence of Tuples and then use fst to extract the Key part of each Tuple. Don’t let Map and map confuse you. Seq.map is just the plain old map function that you know and love.

What we ultimately want out of this is a Set so our last step is to convert our Sequence of Positions into a Set of Positions. With all that written, we can do the following.

{{< highlight fsharp "style=tango" >}}
let Available = FindPositions None
{{< /highlight >}}

Because we’re using Sets instead of Lists, our IsWin function is a little simpler than in the previous example. We no longer have to write functions to decide if a list contains another list. We can simply use the Subset behavior of Sets.

{{< highlight fsharp "style=tango" >}}
let IsWin (player: Player) (board: Board) =
    let playersSquares = 
        board |> FindPositions (Some player)
    wins 
    |> Set.exists (fun win -> win.IsSubsetOf playersSquares)
{{< /highlight >}}

And our IsDraw function is also simple. If a given position isn’t a win then it’s easy to check if it’s a draw, simply check if there is no where left to move. Note that both the Available and IsDraw functions use “Point-Free Syntax“.

{{< highlight fsharp "style=tango" >}}
let IsDraw = Available >> Set.isEmpty
{{< /highlight >}}

Another helper function now, for a given player we need a way of knowing the opponent, this will allow us to toggle back and forth between players as we search recursively for the best move.

{{< highlight fsharp "style=tango" >}}
let Opponent = function
    | X -> O
    | O -> X
{{< /highlight >}}

Actually making a move is just a matter of adding a mapping from a Position to a Player (or Some Player as Options would have us say).

{{< highlight fsharp "style=tango" >}}
let Move (player: Player) (position: Position) (board: Board) =
    board.Add(position, Some player)
{{< /highlight >}}

Note that the last argument to Move is the Board, and the function also returns a board. This allows us to use the following syntax.

{{< highlight fsharp "style=tango" >}}
let pos = 
    NewBoard
    |> Move X TopLeft
    |> Move O TopMiddle
    |> Move X Center
    |> Move O TopRight
{{< /highlight >}}

After running the code above, pos will be a board containing X in the Top Left and Center and O in the Top Middle and Top Right. The rest of the positions will be empty.

And now, the big step, the functions that actually do the work of funding best moves for a given Player faced with a given Board.

{{< highlight fsharp "style=tango" >}}
let rec Score (player: Player) (board: Board) =
    if (IsWin player board) then board |> Available |> Set.count |> (+) 1
    else if (IsDraw board) then 0
    else
        let opponent = Opponent player
        let opponentsBestMove = BestMove opponent board
        let newBoard = Move opponent opponentsBestMove board
        -Score opponent newBoard

and BestMove (player: Player) (board: Board): Position =
    Available board
    |> Set.toList
    |> List.maxBy (fun m -> Score player (board.Add(m, Some player))) 
{{< /highlight >}}

Apart from changes to use Sets and the new Move Syntax, this code is like that in the previous post.

There is one small but significant change. If the earlier code had a choice between a definite win in 1 move or in 2 moves it didn’t care which it took, both were definite wins. This led to it passing up winning moves and winning on the next move instead. It looked a little odd.

To solve this we change how we value wins. Instead of assigning a score of 1 to all wins, which we did in the last version of the code, the Score is now based on how many empty squares remain when the game ends. This makes quicker wins more valuable.

I kind of liked the way the previous implementation sometimes seemed to toy with its victim, so I wouldn’t necessarily call this fix an improvement.