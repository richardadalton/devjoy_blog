+++
date = "2013-04-12T17:43:00Z"
title = "C# Is Too Damn Noisy"
tags = ["code", "c#"]
categories = ["thoughts"]
banner = "img/banners/banner-1.jpg"
+++

I am growing increasingly frustrated with C#. I think the reason for that may be my exposure to languages like F#. In many ways my feelings for C# are quite similar to feelings I had about VB.Net when I was first exposed to C#.

It’s taken me a while to figure out what it is that I find irritating about C# and I think I’m ready to call it. The problem with C# is exactly the same problem I had with VB.Net, it’s too damn noisy.

Take a look at this [code from Martin Rosén-Lidholm](http://martinsaspects.blogspot.ie/2011/01/conways-game-of-life-code-kata.html)

This is clearly a good programmer who cares a lot about writing good code. He went through not one but two approaches that he discarded before settling on a somewhat functional approach. His journey mirrors almost exactly my experience of playing with Conway’s Game of Life.

His ultimate solution in C# is almost identical to [my F# approach](http://www.devjoy.com/2012/07/getting-functional-with-f-and-the-game-of-life/) which was itself based on Vagif Abilov’s approach as presented at NDC Oslo in 2012. The similarity in approach makes it an excellent case study on the relative noise levels in C# and F#.

> Noise is code that is not related to the problem being solved, but is instead related to the language, tools, frameworks etc being used.

The first thing I note in Martin’s solution is that Coordinate implements IEquatable, which means two functions called Equals and one called GetHashCode.

By any definition, implementing IEquatable and getting hash codes is not about Conway’s Game Of Life, so, this is noise.

Now, ok, maybe you can build Game Of Life without implementing IEquatable, but I’m sure Martin does it simply because it’s the “right” thing to do. He’s using LINQ and IEnumerable and for historic reasons this is something you do in C# for performance reasons or whatever. But, to me, it’s noise.

In the F# version there is none of that.

Let’s look at all that text in blue in Martin’s example.

{{< highlight csharp "style=tango" >}}
‘public class’
‘public void’
‘new’
‘private readonly’
‘params’
‘return’
‘private static’
‘private readonly int’
‘public override bool’
‘unchecked’
{{< / highlight >}}

And all those
{{< highlight csharp "style=tango" >}}
{
curly brackets
}
{{< / highlight >}}

oh the humanity.

I haven’t even mentioned all of the explicit specifying of types for fields, for arguments and for return types.

All this is just the pieces of code that have nothing to do with the problem domain. Let’s look at the code that actually does something.

Here’s Martin’s code for creating the next generation

{{< highlight csharp "style=tango" >}}
public Grid CreateNextGeneration() {
    var keepAliveCoordinates = _aliveCoordinates
        .Where(c => GetNumberOfAliveNeighboursOf(c) == 2 || GetNumberOfAliveNeighboursOf(c) == 3);

    var reviveCoordinates = _aliveCoordinates
        .SelectMany(GetDeadNeighboursOf)
        .Where(c => GetNumberOfAliveNeighboursOf(c) == 3);

        return new Grid(keepAliveCoordinates.Union(reviveCoordinates).ToArray());
}
{{< / highlight >}}

Here’s the same function from the F# solution.

{{< highlight fsharp "style=tango" >}}
let nextGenerationOf existingCells =
       cellsThatSurviveFrom existingCells
       @
       cellsThatAreAddedTo existingCells
{{< / highlight >}}

OK, perhaps that’s not the fairest comparison, Martin mentions in a comment that he could push some of that LINQ out into helper functions, but even still, there’s a point here I think.

Is the F# code perfect? No, I dislike the following …

{{< highlight fsharp "style=tango" >}}
let neighboursOfLiveCells pattern = 
    List.map (fun x -> neighboursOf x) pattern 
    |> flattenList 
    |> Set.ofList |> Set.toList
{{< / highlight >}}

This was the first F# program I wrote, and maybe if I take another swing at it I can tidy that up a bit, but still, this was the first F# program I wrote, and that function is all that annoys me. That’s ‘noise’ in the F# context.

So far, I’m finding that the noise generally in F# code is much lower than in C# code. I don’t know if that will hold true as I scale up the problems I’m working on, but I see no reason why it wouldn’t. There is a lot of ceremony in C# that really only distracts from the work that the code is supposed to be doing.

None of this is a reflection on Martin, he clearly knows his stuff and cares a great deal about the code he writes. I am using his blog as an example for no other reason than it’s a great example of C# code solving a problem in the same way as some F# code that I happen to have to hand.