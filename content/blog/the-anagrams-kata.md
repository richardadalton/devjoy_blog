+++
date = "2012-11-11T00:00:00Z"
title = "The Anagrams Kata"
tags = ["testing", "tdd", "c#"]
categories = ["how to"]
banner = "img/banners/permutations.png"
aliases = [
    "/2012/11/the-anagrams-kata/",
]
+++

The following is my C# implementation of the Anagrams Kata as described on cyber-dojo.com

Write a program to generate all potential
anagrams of an input string.

For example, the potential anagrams of “biro” are

{{< highlight text "style=tango" >}}
biro bior brio broi boir bori
ibro ibor irbo irob iobr iorb
rbio rboi ribo riob roib robi
obir obri oibr oirb orbi orib
{{< / highlight >}}

Let’s write a test.

{{< highlight csharp "style=tango, hl_lines=5" >}}
[Test]
public void NoCharacters()
{
    var expected = new List<string> {""};
    Assert.That(Anagrams.Of(""), Is.EqualTo(expected));
}
{{< / highlight >}}

As is almost always the case, we start with the degenerate case of an empty string. We don’t do this solely to ensure the degenerate case is handled (that’s part of it), but with this first test we’re thinking about the API of our new code.

You can see from the highlighted line that the Function we’re writing will be a static method called ‘Of’ on a class called ‘Anagrams’, which allows us the somewhat fluent usage ‘Anagrams.Of(s)’. This is simple Test Driven Design, we’re writing tests as if the desired code already exists, which in theory gives us the best API we could possibly hope for.

Let’s make that test pass. We’ll need a Class, a Method, and we’ll return a literal that satisfies the test.

{{< highlight csharp "style=tango" >}}
public class Anagrams
{
    public static List<string> Of(string s)
    {
        return new List<string> {""};
    }
}
{{< / highlight >}}

We have GREEN. We might have been tempted to just return s and be done with it. It would have passed the test, and it would have handled the case of one character strings also. Thinking about passing tests that we haven’t even written yet suggests we’re getting ahead of ourselves. Let’s write the test for the one character example.

{{< highlight csharp "style=tango" >}}
[Test]
public void OneCharacter()
{
    var expected = new List<string> { "A" };
    Assert.That(Anagrams.Of("A"), Is.EqualTo(expected));
}
{{< / highlight >}}

Now we’ve got a failing test, let’s get it passing. We can return ‘s’ as mentioned above, and both our tests would pass. It’s ok to take these kinds of steps, if a test fails unexpectedly we can revert the changes and try again a little more slowly.

For the sake of the exercise, let’s go slow for now and look at a technique that will serve us well later when things are more complicated. We’ll isolate the existing working code, make the new test pass with a branch in the code, then try to eliminate the branch. Here’s a way of getting both tests passing.

{{< highlight csharp "style=tango" >}}
public static List<string> Of(string s)
{
    if(s == "")
        return new List<string> {""};
 
    return new List<string>{"A"};
}
{{< / highlight >}}

The ‘if’ statement handles the empty string case, so our new code doesn’t break it. We use a literal value “A” to get the new test passing as simply as possible. This leaves us with duplication between the literal value “A” in the test and the same value in the code. We can remove the duplication by generalising the solution for One character, returning s is the more general form of returning “A”.

{{< highlight csharp "style=tango" >}}
public static List<string> Of(string s)
{
    if(s == "")
        return new List<string> {""};
 
    return new List<string>{s};
}
{{< / highlight >}}

It’s clear now that returning s will make both our tests pass, so let’s refactor.

{{< highlight csharp "style=tango" >}}
public static List<string> Of(string s)
{
    return new List<string>{s};
}
{{< / highlight >}}

Time for another test. How about two characters?

{{< highlight csharp "style=tango" >}}
[Test]
public void TwoCharacters()
{
    var expected = new List<string> { "AB", "BA" };
    Assert.That(Anagrams.Of("AB"), Is.EqualTo(expected));
}
{{< / highlight >}}

Let’s repeat the same trick of protecting the existing working code with an if statement, and making the test pass using the simplest possible implementation.

{{< highlight csharp "style=tango" >}}
public static List<string> Of(string s)
{
    if(s.Length<=1)
        return new List<string>{s};

    return new List<string>
               {
                   "AB", 
                   "BA"
               };
}
{{< / highlight >}}

We’re back to having duplication between our test and our code, let’s break that.

{{< highlight csharp "style=tango" >}}
public static List<string> Of(string s)
{
    if(s.Length<=1)
        return new List<string>{s};

    return new List<string>
               {
                   s, 
                   s.Substring(1,1) + s.Substring(0,1)
               };
}
{{< / highlight >}}

That’ll work for any strings from 0 to 2 characters. Next test, three characters.

{{< highlight csharp "style=tango" >}}
[Test]
public void ThreeCharacters()
{
    var expected = new List<string> { "ABC", "ACB", "BAC", "BCA", "CAB", "CBA" };
    Assert.That(Anagrams.Of("ABC"), Is.EqualTo(expected));
}
{{< / highlight >}}

And to make this pass? You should know the ropes by now. Isolate existing code, and hard code a solution.

{{< highlight csharp "style=tango" >}}
public static List<string> Of(string s)
{
    if (s.Length <= 1)
        return new List<string> { s };

    if(s.Length == 2)
    {
        return new List<string>
               {
                   s, 
                   s.Substring(1,1) + s.Substring(0,1)
               };
    }

    return new List<string>
               {
                   "ABC", 
                   "ACB", 
                   "BAC", 
                   "BCA", 
                   "CAB", 
                   "CBA"
               };
}
{{< / highlight >}}

There’s a pattern emerging here. Lets split up the strings to make it more clear.

{{< highlight csharp "style=tango" >}}
return new List<string>
           {
               "A" + "BC", 
               "A" + "CB", 
               "B" + "AC", 
               "B" + "CA", 
               "C" + "AB", 
               "C" + "BA"
           };
{{< / highlight >}}

We can replace the first character on each row.

{{< highlight csharp "style=tango" >}}
return new List<string>
           {
               s.Substring(0,1) + "BC", 
               s.Substring(0,1) + "CB", 
               s.Substring(1,1) + "AC", 
               s.Substring(1,1) + "CA", 
               s.Substring(2,1) + "AB", 
               s.Substring(2,1) + "BA"
           };
{{< / highlight >}}

What has that acheived? We still have literal values on each line. Well, we do, but they look famililar. “BC” and “CB” look like the anagrams of “BC”. Same applies to “AC”, “CA”, and “AB”, “BA”. We’ve discovered that part of the job of generating anagrams for three characters is to generate anagrams for two characters. Let’s see if we can make that explicit.

{{< highlight csharp "style=tango" >}}
return new List<string>
           {
               s.Substring(0,1) + Anagrams.Of("BC")[0], 
               s.Substring(0,1) + Anagrams.Of("BC")[1], 
               s.Substring(1,1) + "AC", 
               s.Substring(1,1) + "CA", 
               s.Substring(2,1) + "AB", 
               s.Substring(2,1) + "BA"
           };
{{< / highlight >}}

If we get Anagrams.Of(“BC”) we’ll get two results. The first will be “BC” the second will be “CB”. We can repeat this change for the remaining literal values.

{{< highlight csharp "style=tango" >}}
return new List<string>
           {
               s.Substring(0,1) + Anagrams.Of("BC")[0], 
               s.Substring(0,1) + Anagrams.Of("BC")[1], 
               s.Substring(1,1) + Anagrams.Of("AC")[0], 
               s.Substring(1,1) + Anagrams.Of("AC")[1], 
               s.Substring(2,1) + Anagrams.Of("AB")[0], 
               s.Substring(2,1) + Anagrams.Of("AB")[1]
           };
{{< / highlight >}}

Hang on, we still have literal values on all 6 lines. Are we making any progress at all? or are we just moving the problem around? We have made progress. We’ve actually eliminated half the literals. Now we’re almost in a position to remove those literals completely. Remember the original string ‘s’ has a value of “ABC”, so to get the “BC” of the first two lines, we need to drop the first character (“A”) from “ABC”.

Time to do a little more TDDing, let’s pretend we have the function we need.

{{< highlight csharp "style=tango" >}}
return new List<string>
           {
               s.Substring(0,1) + Anagrams.Of(DropCharacter(s,0))[0], 
               s.Substring(0,1) + Anagrams.Of(DropCharacter(s,0))[1], 
               s.Substring(1,1) + Anagrams.Of("AC")[0], 
               s.Substring(1,1) + Anagrams.Of("AC")[1], 
               s.Substring(2,1) + Anagrams.Of("AB")[0], 
               s.Substring(2,1) + Anagrams.Of("AB")[1]
           };
{{< / highlight >}}

This won’t compile, we don’t have the DropCharacter function. Let’s get the test passing in the simplest possible way.

{{< highlight csharp "style=tango" >}}
private static string DropCharacter(string s, int index)
{
    return "BC";
}
{{< / highlight >}}

Strictly speaking I’ve skipped a step there. What I would actually do is create the function so that everything compiles, but have it return the wrong value, so that I have a failing test. Then by having the function return “BC” I get the test to Green. This reassures me that my tests are actually joined up to the code.

We’re still have literal value duplication between our tests and our code, but let’s get DropCharacter fully working before we tackle that. Right now it works if we want to drop the first character (“A”). Let’s try to expand it’s use.

{{< highlight csharp "style=tango" >}}
return new List<string>
           {
               s.Substring(0,1) + Anagrams.Of(DropCharacter(s,0))[0], 
               s.Substring(0,1) + Anagrams.Of(DropCharacter(s,0))[1], 
               s.Substring(1,1) + Anagrams.Of(DropCharacter(s,1))[0], 
               s.Substring(1,1) + Anagrams.Of(DropCharacter(s,1))[1], 
               s.Substring(2,1) + Anagrams.Of("AB")[0], 
               s.Substring(2,1) + Anagrams.Of("AB")[1]
           };
{{< / highlight >}}

This will give us a failing test. The next failing test doesn’t always have to be a new test. We actually have all of the tests we need. What has happened here is that we’ve tried to make one part of our code more general, in doing so we’ve identified a hard coding issue elsewhere in the code. We are effectively test driving the ‘DropCharacter’ function by proxy. Our ‘Anagrams.Of’ function is using ‘DropCharacter’ in ways that it can’t handle.

You should know the drill for getting back to green by now, protect the working code with an ‘if’ and get the new test passing as simply as possible.

{{< highlight csharp "style=tango" >}}
private static string DropCharacter(string s, int index)
{
    if(index == 0)
        return "BC";

    return "AC";
}
{{< / highlight >}}

Let’s sort out the final case for ‘DropCharacter’ and then see where we stand.

{{< highlight csharp "style=tango" >}}
return new List<string>
           {
               s.Substring(0,1) + Anagrams.Of(DropCharacter(s,0))[0], 
               s.Substring(0,1) + Anagrams.Of(DropCharacter(s,0))[1], 
               s.Substring(1,1) + Anagrams.Of(DropCharacter(s,1))[0], 
               s.Substring(1,1) + Anagrams.Of(DropCharacter(s,1))[1], 
               s.Substring(2,1) + Anagrams.Of(DropCharacter(s,2))[0], 
               s.Substring(2,1) + Anagrams.Of(DropCharacter(s,2))[1]
           };
{{< / highlight >}}


To get this passing we’ll use another ‘if’

{{< highlight csharp "style=tango" >}}
private static string DropCharacter(string s, int index)
{
    if(index == 0)
        return "BC";

    if(index == 1)
        return "AC";

    return "AB";
}
{{< / highlight >}}


Time for some refactoring now. We need to eliminate these literal values once and for all. Dropping the first and last characters is easy.

{{< highlight csharp "style=tango" >}}
private static string DropCharacter(string s, int index)
{
    if(index == 0)
        return s.Substring(1,2);

    if(index == 1)
        return "AC";

    return s.Substring(0,2);
}
{{< / highlight >}}


Of course we’ve hard coded the parameters to Substring, which means we’re still tied to strings of three characters. One step at a time. Let’s sort out the case where we’re dropping the middle character, and then see if we can generalise for any string length.

{{< highlight csharp "style=tango" >}}
if(index == 1)
    return "A" + "C";
{{< / highlight >}}


The letter “B” has been dropped, “A” and “C” represent the parts of the string before and after the dropped letter. So, we split the string and deal with each part seperately.

{{< highlight csharp "style=tango" >}}
if(index == 1)
    return s.Substring(0,1) + s.Substring(2,1);
{{< / highlight >}}


Here’s where we stand.

{{< highlight csharp "style=tango" >}}
private static string DropCharacter(string s, int index)
{
    if(index == 0)
        return s.Substring(1,2);

    if(index == 1)
        return s.Substring(0,1) + s.Substring(2,1);

    return s.Substring(0,2);
}
{{< / highlight >}}


We’ve eliminated the dependency on literal strings, but we’ve replaced it with a dependency on strings a of particular length. That’s progress of sorts. We have the string ‘s’ and the position of the dropped character ‘index’ to work with. We need to trace these hard coded values back to these two inputs. This can be a little tricky. Just because you know that index will have a value of 1 under certain circumstances, doesn’t mean that every instance of the value 1 can be replaced with index.

Let’s start with the portion of the string, before the dropped character. There are two places in the function where that piece of string is referred to, and by using the ‘index’ variable we can eliminate some of the hard coding of the number of characters.

{{< highlight csharp "style=tango, hl_lines=7 9" >}}
private static string DropCharacter(string s, int index)
{
    if(index == 0)
        return s.Substring(1,2);

    if(index == 1)
        return s.Substring(0,index) + s.Substring(2,1);

    return s.Substring(0,index);
}
{{< / highlight >}}


We can introduce a ‘before’ variable that makes this explicit.

{{< highlight csharp "style=tango" >}}
private static string DropCharacter(string s, int index)
{
    var before = s.Substring(0, index);

    if (index == 0)
        return s.Substring(1,2);

    if(index == 1)
        return before + s.Substring(2,1);

    return before;
}
{{< / highlight >}}


Now we need to deal with the portion of the string that comes after the dropped character. Again we have two examples of this in the code. We can see that the first parameter to Substring, the starting position of the substring seems to be ‘index + 1’. Let’s plug that in.

{{< highlight csharp "style=tango, hl_lines=6 9" >}}
private static string DropCharacter(string s, int index)
{
    var before = s.Substring(0, index);

    if (index == 0)
        return s.Substring(index + 1, 2);

    if(index == 1)
        return before + s.Substring(index + 1, 1);

    return before;
}
{{< / highlight >}}


The second parameter to Substring isn’t quite so simple, it depends on both the index of the dropped character and the length of the original string. Nonetheless it’s reasonably easy to figure out.

{{< highlight csharp "style=tango" >}}
private static string DropCharacter(string s, int index)
{
    var before = s.Substring(0, index);

    if (index == 0)
        return s.Substring(index + 1, s.Length - (index + 1));

    if(index == 1)
        return before + s.Substring(index + 1, s.Length - (index + 1));

    return before;
}
{{< / highlight >}}


For clarity, let’s introduce another variable to really make this all obvious.

{{< highlight csharp "style=tango" >}}
private static string DropCharacter(string s, int index)
{
    var before = s.Substring(0, index);
    var after = s.Substring(index + 1, s.Length - (index+1));

    if (index == 0)
        return after;

    if(index == 1)
        return before + after;

    return before;
}
{{< / highlight >}}


And, we can simplify this, as follows

{{< highlight csharp "style=tango" >}}
private static string DropCharacter(string s, int index)
{
    var before = s.Substring(0, index);
    var after = s.Substring(index + 1, s.Length - (index+1));

    return before + after;
}
{{< / highlight >}}


Our final task is to turn our attention back to the ‘Anagrams.Of’ function and sort it out.

{{< highlight csharp "style=tango, hl_lines=17-22" >}}
public static List<string> Of(string s)
{
    if (s.Length <= 1)
        return new List<string> { s };

    if(s.Length == 2)
    {
        return new List<string>
               {
                   s, 
                   s.Substring(1,1) + s.Substring(0,1)
               };
    }

    return new List<string>
               {
                   s.Substring(0,1) + Anagrams.Of(DropCharacter(s,0))[0], 
                   s.Substring(0,1) + Anagrams.Of(DropCharacter(s,0))[1], 
                   s.Substring(1,1) + Anagrams.Of(DropCharacter(s,1))[0], 
                   s.Substring(1,1) + Anagrams.Of(DropCharacter(s,1))[1], 
                   s.Substring(2,1) + Anagrams.Of(DropCharacter(s,2))[0], 
                   s.Substring(2,1) + Anagrams.Of(DropCharacter(s,2))[1]
               };
}
{{< / highlight >}}


Those six lines of code that create the six anagrams of a three character word look promising. There’s a reasonably obvious loop happening there.

The first thing we need to do is get rid of the list initialiser code, so that we can add items to the list using a loop.

{{< highlight csharp "style=tango" >}}
var anagrams = new List<string>();
anagrams.Add(s.Substring(0, 1) + Anagrams.Of(DropCharacter(s, 0))[0]);
anagrams.Add(s.Substring(0, 1) + Anagrams.Of(DropCharacter(s, 0))[1]);
anagrams.Add(s.Substring(1, 1) + Anagrams.Of(DropCharacter(s, 1))[0]);
anagrams.Add(s.Substring(1, 1) + Anagrams.Of(DropCharacter(s, 1))[1]);
anagrams.Add(s.Substring(2, 1) + Anagrams.Of(DropCharacter(s, 2))[0]);
anagrams.Add(s.Substring(2, 1) + Anagrams.Of(DropCharacter(s, 2))[1]);
return anagrams;
{{< / highlight >}}


Now let’s get a loop working for us and delete some of this code.

{{< highlight csharp "style=tango" >}}
var anagrams = new List<string>();

for (int i = 0; i < 3; i++ )
{
    anagrams.Add(s.Substring(i, 1) + Anagrams.Of(DropCharacter(s, i))[0]);
    anagrams.Add(s.Substring(i, 1) + Anagrams.Of(DropCharacter(s, i))[1]);                
}

return anagrams;
{{< / highlight >}}


We can nest another loop to reduce things even further

{{< highlight csharp "style=tango" >}}
for (int i = 0; i < 3; i++ )
    for (var j = 0; j < 2; j++)
        anagrams.Add(s.Substring(i, 1) + Anagrams.Of(DropCharacter(s, i))[j]);
{{< / highlight >}}


The upper bounds of both loop are hardcoded. Let’s get rid of those.

{{< highlight csharp "style=tango" >}}
for (int i = 0; i < s.Length; i++ )
    for (var j = 0; j < s.Length - 1; j++)
        anagrams.Add(s.Substring(i, 1) + Anagrams.Of(DropCharacter(s, i))[j]);
{{< / highlight >}}


It took a while to get here, but now we can tidy up the ‘Anagrams.Of’ function.

{{< highlight csharp "style=tango" >}}
public static List<string> Of(string s)
{
    if (s.Length <= 1)
        return new List<string> { s };

    var anagrams = new List<string>();

    for (var i = 0; i < s.Length; i++ )
        for (var j = 0; j < s.Length - 1; j++)
            anagrams.Add(s.Substring(i, 1) + Anagrams.Of(DropCharacter(s, i))[j]);

    return anagrams;
}
{{< / highlight >}}


And that’s it. Here’s the full listing of Tests and Code with a little bit of extra tweeking.

{{< highlight csharp "style=tango" >}}
using System.Collections.Generic;
using NUnit.Framework;

namespace AnagramKata
{
    [TestFixture]
    public class AnagramTests
    {
        [Test]
        public void NoCharacters()
        {
            var expected = new List<string> { "" };
            Assert.That(Anagrams.Of(""), Is.EqualTo(expected));
        }

        [Test]
        public void OneCharacter()
        {
            var expected = new List<string> {"A"};
            Assert.That(Anagrams.Of("A"), Is.EqualTo(expected));
        }

        [Test]
        public void TwoCharacters()
        {
            var expected = new List<string> { "AB", "BA" };
            Assert.That(Anagrams.Of("AB"), Is.EqualTo(expected));
        }

        [Test]
        public void ThreeCharacters()
        {
            var expected = new List<string> { "ABC", "ACB", "BAC", "BCA", "CAB", "CBA" };
            Assert.That(Anagrams.Of("ABC"), Is.EqualTo(expected));
        }
    }

    public class Anagrams
    {
        public static List<string> Of(string s)
        {
            if (s.Length <= 1)
                return new List<string> { s };

            var anagrams = new List<string>();

            for (var i = 0; i < s.Length; i++ )
            {
                var root = s.Substring(i, 1);
                var rest = DropCharacter(s, i);
                for (var j = 0; j < s.Length - 1; j++)
                    anagrams.Add(root + Anagrams.Of(rest)[j]);
            }

            return anagrams;
        }

        private static string DropCharacter(string s, int index)
        {
            return s.Substring(0, index) + s.Substring(index + 1, s.Length - (index+1));
        }
    }
}
{{< / highlight >}}

