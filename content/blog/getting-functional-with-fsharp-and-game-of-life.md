+++
date = "2012-07-15T16:00:00Z"
title = "Getting Functional with F# and The Game Of Life"
tags = ["code", "functional programming", "f#"]
categories = ["how to"]
banner = "img/banners/fsharp.png"
aliases = [
    "/2012/07/getting-functional-with-f-and-the-game-of-life/",
]
+++

One session at NDC that really kicked my grasp of functional programming up a few notches was Vagif Abilov’s discussion of Conway’s Game Of Life using F#.

I’m not going to rehash the rules of Game Of Life here, if you aren’t familiar with them then [read this](http://en.wikipedia.org/wiki/Conway's_Game_of_Life).

Vagif’s source code is on [github](https://github.com/object/ConwayGame) and his slides are on [slideshare](http://www.slideshare.net/VagifAbilov/f-in-action-playing-functional-conways-game-of-life). His stuff is well worth a look, but don’t look yet.

If you want to really play along with the spirit of this post, stop reading now, and go and code up Conway’s game of life in the object oriented language of your choice. I’ll be here when you get back.

Vagif opened his session with a brief discussion of what an Object Oriented/C# solution might look like. It might for example include a class to represent a cell, perhaps with a state (isAlive) property. It may contain references to it’s neighbours which would themselves be cells. It may even contain the rules for when a cell survives, dies or comes to life. He mentioned one or two very over engineered examples that can be found on the interwebs, those are also worth a look. A cautionary tale on being “Pattern Happy”.

I had recently toyed around with Game of Life for an hour or two in C# so I was hooked from the start. Within 5 minutes of the session starting, I recognised two very simple but fundamental flaws in the way I had approached the problem.

I was thinking in terms of a preallocated board containing cells that are either alive or dead.
I was thinking of a process that modifies the state of the board by killing existing cells and bringing new ones to life.

On the face of it it’s hard to argue with either of those ideas, and it’s hard to see how they could cause any serious problems.

Let’s take them in turn.

A preallocated board brings with it the inbuilt limitation of it’s dimensions (unless we also allow it to be resized). It also requires us to allocate memory for cells that only *might* come to life at some point. We could simply store a list of cells that are alive, and allow some mechanism for figuring out if two cells are neighbours.

A process that modifies the state of the board by killing cells or bringing them to life might work, but we probably don’t want to turn on and off cells while we’re still looking at whether others should live or die, so realistically we would create a copy of the board, use one to make decisions and the other to hold the new state.

Also, since we’re no longer holding on to Dead Cells, we don’t need to worry about killing cells, we simply don’t bother including them in the list of cells that will be in the next generation.

The algorithm for calculating the next generation from a given list of cells becomes trivial

Take the existing list of cells and decide which should survive, we’ll call this list the ‘Survivors’.
Determine the “Dead” neighbours of the existing list of cells and decide who should come to life, we’ll call this the ‘Newborns’
The next generation is created by concatenating the Survivors and the Newborns to create a new list.

I came away from Vagif’s session understanding his approach in about as much detail as I’ve described above. I didn’t know F# well enough to code it immediately, but I decided to see if I could figure it out without peeking at Vagif’s code.

I thought a quick comparison of my results and Vagif’s original code might prove useful to someone learning F# hence, this post.

### Representing the “Board”
Let’s start with how we’ll represent the pattern of cells. The way Game of Life is generally described conjures up images of something akin to a chess board, perhaps for this reason it’s not uncommon to find concepts like ‘Board’ in the code, often in the form of a two dimensional array. I preferred to follow Vagif’s lead and went with a list of tuples representing (x, y) coordinates.

We can have some fun here with how we lay out our code. Look at the following two lines, they are functionally identical, but the second one actually illustrates the relative positions of the coordinates we’re defining. We’ll see something similar to this a little later.

{{< highlight fsharp "style=tango" >}}
let liveCells = [(-1,0); (0, -1); (0,0); (0,1); (1,0)]
 
let liveCells = [            (-1,0); 
                    (0, -1); (0,0); (0,1); 
                             (1,0) ]
{{< / highlight >}}

Now that we know what we’re dealing with, let’s write some functions to manipulate our pattern of cells.

To check if a specific cell (i.e. coordinate) is alive or dead, we just have to find out if it’s in our list of live cells.

{{< highlight fsharp "style=tango" >}}
let isAlive pattern cell = 
    List.exists (fun elem -> elem = cell) pattern
 
let isDead pattern cell =
    isAlive pattern cell |> not
{{< / highlight >}}

isDead is just a negation of isAlive, so it’s not all that interesting. isAlive accepts a pattern (i.e. a list of cells) and a specific cell to check. We can call it as follows:

{{< highlight fsharp "style=tango" >}}
> isAlive liveCells (-1,0);;
val it : bool = true
{{< / highlight >}}

Vagif’s isAlive is a little cleaner than mine, and he doesn’t bother with an isDead. We’ll see why later.

{{< highlight fsharp "style=tango" >}}
let isAlive pattern cell =
    pattern |> List.exists ((=) cell)
{{< / highlight >}}

### Find The Neighbours
A key part of The Game of Life is figuring out who are the neighbours of a given cell, and determining how many of those neighbours are alive. Let’s start with determining the neighbours for a specific cell. If you imagine the cell at the center of a 3X3 grid, you can see that any cell has 8 neighbours. Here’s my function for returning the 8 neighbours of a given cell.

{{< highlight fsharp "style=tango" >}}
let neighboursOf (x, y) =
    [
      (x-1, y-1);   (x-1, y);   (x-1, y+1);
      (x, y-1);                 (x, y+1);
      (x+1, y-1);   (x+1, y);   (x+1, y+1);
    ]
{{< / highlight >}}

Note again, I could have laid this out as a simple list, but by splitting it over three lines and using a bit of creative whitespace, I can illustrate the relative positions of the 8 neighbours. I had originally been trying to use a nested loop, but couldn’t figure out the syntax and went with this instead. Vagif’s code uses the nested loop approach.

{{< highlight fsharp "style=tango" >}}
let neighbours (x, y) =
    [ for i in x-1..x+1 do
      for j in y-1..y+1 do
      if not (i = x && j = y) then yield (i,j) ]
{{< / highlight >}}

He does need an if statement to stop a cell registering as a neighbour of itself. I don’t need that in my version. In his favour however is the fact that his approach is much easier to scale up to a third dimension.

That little exercise in finding the neighbours was the first glimpse I had of the power of functional programming. I couldn’t figure out the syntax to state HOW to figure out the neighbours of a cell, so I just stated what the end result should LOOK LIKE. And it worked.

How would we get a list of the neighbours of a cell that are alive? Simples…

{{< highlight fsharp "style=tango" >}}
let liveNeighbours pattern cell =
    neighboursOf cell |> List.filter (isAlive pattern)
{{< / highlight >}}

We pipe (‘|>’) the list of neighbours for a cell to a List.filter operation, where we filter using the isAlive function we defined above. This function is basically identical to Vagif’s.

I wrote a separate function for counting the number of live neighbours for a cell, which just gets the length of the list created here. Vagif doesn’t bother with this step.

{{< highlight fsharp "style=tango" >}}
let numberOfLiveNeighbours pattern cell =
    liveNeighbours pattern cell |> List.length
{{< / highlight >}}

Deciding whether a cell should remain alive for the next generation is just a question of seeing if it has either 2 or 3 neighbours that are alive.

{{< highlight fsharp "style=tango" >}}
let cellStaysAlive pattern cell = 
    numberOfLiveNeighbours pattern cell = 2 || numberOfLiveNeighbours pattern cell= 3
{{< / highlight >}}

Once we’ve written that function for checking an individual cell, we can use it to filter all the existing live cells to create our ‘Survivors’ list.

{{< highlight fsharp "style=tango" >}}
let cellsThatSurviveFrom cells =
    cells 
    |> List.filter (cellStaysAlive cells) 
{{< / highlight >}}

As we know finding out which of the existing cells that survive is one of the two things we need to figure out, all that remains is to find out which new cells will come to life. Unfortunately finding survivers is the easier of the two. Finding new cells involves a few steps.

Find all neighbours for the existing live cells
Filter the list of neighbours so that it excludes any cells that are already alive
Count the number of existing live neighbours there are for each of the dead cells identified above
Create a new list containing only dead neighbours of existing cells, that themselves have three existing neighbours

Let’s take this one step at a time, first lets find all the neighbours for all of the live cells. We do this by applying the neighbourOf function to each cell in whatever pattern is passed in.

{{< highlight fsharp "style=tango" >}}
let neighboursOfLiveCells pattern = 
    List.map (fun x -> neighboursOf x) pattern 
    |> flattenList 
    |> Set.ofList |> Set.toList
{{< / highlight >}}

Mapping the neighbourOf function to the pattern results in a list of lists. We then flatten that list of lists, and remove duplicates. The flattedList method and then converting the list to a set and back to a list.

The code for flattenList is as follows:

{{< highlight fsharp "style=tango" >}}
let rec flattenList ls =
        match ls with
        | [] -> []
        | head::tail -> head @ flattenList tail
{{< / highlight >}}

I did find a way of using the reduce feature of F# to accomplish the same thing, but I already had this way working and so didn’t change it.

The next step is to filter the neighbours so that the list only contains dead cells.

{{< highlight fsharp "style=tango" >}}
let potentialCells pattern =
    pattern 
    |> neighboursOfLiveCells 
    |> List.filter (isDead pattern)
{{< / highlight >}}

Create a function that can determine if a specific cell comes alive.

{{< highlight fsharp "style=tango" >}}
let cellComesAlive pattern cell = 
    numberOfLiveNeighbours pattern cell = 3
{{< / highlight >}}

Filter the list of Neighbours down to cells that should come alive. This is our list of newborns.

{{< highlight fsharp "style=tango" >}}
let cellsThatAreAddedTo existingCells =
    potentialCells existingCells 
    |> List.filter (cellComesAlive existingCells)
{{< / highlight >}}

Vagif used the List.collect method to simplify this whole process. I wasn’t aware of List.collect when I dreamt up that slightly convoluted way of doing things. Note he also passes the isAlive filter through the ‘not’ operator to negate it. In my code I explicitly create an isDead function.

{{< highlight fsharp "style=tango" >}}
let allDeadNeighbours pattern =
    pattern
    |> List.collect neighbours
    |> Set.ofList |> Set.toList
    |> List.filter (not << isAlive pattern)
{{< / highlight >}}

Now, back to our existing idea for an algorithm, we combing the cells that survive with the cells that come alive and that gives us the next generation.

{{< highlight fsharp "style=tango" >}}
let nextGenerationOf existingCells =
        cellsThatSurviveFrom existingCells
        @
        cellsThatAreAddedTo existingCells
{{< / highlight >}}

Overall I’m reasonably happy that I managed to recreate something close to Vagif’s implementation with very little prior knowledge of F#. I’m still a little dubious about the language itself. It can take a bit of hoop jumping to get things done. However some of those problems are down to lack of familiarity with the language. I would be curious to see a similar implementation in another language like Clojure. I’m also keen to go back and try a C# version again, but done in a more functional way.

In the interests of completeness, here’s my full code listing. Vagif’s code is available from [Github](https://github.com/object/ConwayGame).

{{< highlight fsharp "style=tango" >}}
let rec flattenList ls =
        match ls with
        | [] -> []
        | head::tail -> head @ flattenList tail
 
 
let liveCells = [            (-1,0); 
                    (0, -1); (0,0); (0,1); 
                             (1,0) ]
 
 
let isAlive pattern cell = 
    List.exists (fun elem -> elem = cell) pattern
 
let isDead pattern cell =
    isAlive pattern cell |> not



let neighboursOf (x, y) =
    [
      (x-1, y-1);   (x-1, y);   (x-1, y+1);
      (x, y-1);                 (x, y+1);
      (x+1, y-1);   (x+1, y);   (x+1, y+1);
    ]
 
let liveNeighbours pattern cell =
    neighboursOf cell |> List.filter (isAlive pattern)
     
let numberOfLiveNeighbours pattern cell =
    liveNeighbours pattern cell |> List.length




let cellStaysAlive pattern cell = 
    numberOfLiveNeighbours pattern cell = 2 || numberOfLiveNeighbours pattern cell= 3


let cellsThatSurviveFrom cells =
    cells 
    |> List.filter (cellStaysAlive cells) 



let neighboursOfLiveCells pattern = 
    List.map (fun x -> neighboursOf x) pattern 
    |> flattenList 
    |> Set.ofList |> Set.toList


let potentialCells pattern =
    pattern 
    |> neighboursOfLiveCells 
    |> List.filter (isDead pattern)


let cellComesAlive pattern cell = 
    numberOfLiveNeighbours pattern cell = 3


let cellsThatAreAddedTo existingCells =
    potentialCells existingCells 
    |> List.filter (cellComesAlive existingCells)



let nextGenerationOf existingCells =
        cellsThatSurviveFrom existingCells
        @
        cellsThatAreAddedTo existingCells
{{< / highlight >}}