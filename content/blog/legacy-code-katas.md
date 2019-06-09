+++
date = "2013-01-18T12:00:00Z"
title = "Legacy Code Katas"
tags = ["technical-debt", "code", "testing"]
categories = ["thoughts"]
banner = "img/banners/banner-1.jpg"
aliases = [
    "/2013/01/legacy-code-katas/",
]
+++

I like Kata’s, I’ve get a lot out of them, but if I’m truly honest, they don’t really address one area of programming where I think I need practice, and that is in working with Legacy Code. In pondering this problem I came to the conclusion that I need a way of doing deliberate practice for legacy code work, and some variation of the Kata idea seems like it might work.

I should point out that this isn’t an original idea, there’s at least one StackOverflow question from 2009 asking about this exact topic. Nothing seems to have come out of it however. Perhaps there’s a reason for that.

There are practical reasons why Legacy Code Kata’s are difficult. I’m not going to go into them all now, but for starters a traditional Kata can be stated with a few sentences of English text, and the programmer can implement it in any language, with any tool chain.

Legacy Kata’s will likely require us to provide starting code, which immediately makes them harder to create and limits their audience. Special variations would need to be created for each language for example. Might there be a way of describing the problem to be solved in a language independent way? So that the solver creates the starting code? I don’t know.

To get the ball rolling, or illustrate the idea, I’m including here some code, that could form the basis of an exercise/kata. Along with some tasks that should be accomplished with the code. If you have any feedback as to whether there is any potential here for any kind of useful learning/practice then let me know your thoughts.

# Legacy Code Kata

{{< highlight csharp "style=tango" >}}
public class SecurityManager
{
    public static void CreateUser()
    {
        Console.WriteLine("Enter a username");
        var username = Console.ReadLine();
        Console.WriteLine("Enter your full name");
        var fullName = Console.ReadLine();
        Console.WriteLine("Enter your password");
        var password = Console.ReadLine();
        Console.WriteLine("Re-enter your password");
        var confirmPassword = Console.ReadLine();

        if (password != confirmPassword)
        {
            Console.WriteLine("The passwords don't match");
            return;
        }

        if (password.Length < 8)
        {
            Console.WriteLine("Password must be at least 8 characters in length");
            return;
        }

        // Encrypt the password (just reverse it, should be secure)
        char[] array = password.ToCharArray();
        Array.Reverse(array);

        Console.Write(String.Format("Saving Details for User ({0}, {1}, {2})\n",
            username,
            fullName,
            new string(array)));
    }
}
{{< /highlight >}}

Given the code above, do the following:

* Break the dependency on the Console
* Get the password comparison feature under test
* Get the password validation feature under test
* Add a feature to allow different encryption algorithms to be used

Thoughts? Comments? Would a collection of exercises like this be a useful resource, or a waste of time?