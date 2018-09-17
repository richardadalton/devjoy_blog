+++
date = "2013-01-13T12:00:00Z"
title = "I is for Interface"
tags = ["code", "c#"]
categories = ["thoughts"]
banner = "img/banners/banner-1.jpg"
+++

Back in 1995 I sat in a large ballroom to listen to politicians from the North and South of Ireland talk about the peace process, which at that time was far from a sure thing.

A story, told by one of the Unionists stuck in my head. He told of how he had been involved in selling a ceasefire to loyalist paramilitaries and one of the questions that he was faced with was.

“If we go on ceasefire, who’ll shoot the taigues?” (Catholics)

The story was told light heartedly, and it raised a chuckle around the room. The very notion of someone failing to understand that the whole point of a ceasefire is that the need to shoot people has gone away.

This week I tweeted a far less important question than the life and death decisions of paramilitaries.

Should we continue with the ‘I’ prefix for interfaces, or should be drop it?

The reason why we ask these questions is that over time the reasons why conventions exist can “go away”.

The ‘I’ in interface names is a visual cue, that is all. There is no technical reason for it to be there, and so, there’s every chance that in time it may go away.

Aha! you say. There’s no technical reason why I should call a list of customers ‘Customers’, the compiler would be equally happy with ‘c’, but meaningful variable names are here to stay.

Well, there’s a difference here and it’s worth exploring. Some cues are iconic, they have something about them that make them instantly and intuitively understood on some level. Using meaningful variable names, and plural variable names to represent collections are iconic cues.

Others cues exist and work purely because of convention. Hungarian Notation and the I in Interface names would be an example of these. Things that hang around purely out of habit or convention are far more vulnerable than things that are iconic.

You can move from iconic to convention. At one time using a picture of a 3.5″ floppy disk to represent save was an iconic image, the meaning was instantly recognisable. Now, we have a generation of users clicking happily on that image, despite having never seen much less used a floppy disk.

But I digress.

The point is, the ‘I’ in interface names exists for no other reason than convention. It is not iconic, and it is not technically needed. So, why don’t we all just get on with it and scrap it already?

Debates about the ‘I’ in interfaces or the ‘var’ in variable declarations often come down to us believing that we ‘need’ things to stay the same. When programmers push back against the use of ‘var’ it is because they believe earnestly that they need to know the type of a variable at a glance, at the moment of declaration. Even hovering the mouse over it is too big a price.

When programmers cling to the ‘I’ in interface names it is usually because they earnestly believe that they need to know at a glance whether something is an interface or a class.

When the debate heats up it is often resolved along the lines of…”Well, having the ‘I’ doesn’t do any harm, and it’s just extra information, and extra information is always good. So, misgivings aside, we’ll keep it.

But what if the ‘I’ is doing harm? What if explicitly specifying type is doing harm? The debate rarely gets that far because it’s hard to understand why a little extra context could ever be a bad thing.

If a little extra context is inherently good, why didn’t we continue using Hungarian notation? Why don’t we prefix Abstract classes with ‘A’?

And if we need to provide extra context, why don’t we do it consistently? We suffix the word Exception to Exceptions, and Attribute to Attributes, why not suffix the word Interface to Interfaces? That would actually give a more meaningful name. Loggers could implement the ‘LoggingInterface’.

I was once steadfastly in the anti-var camp. I argued convincingly (I think) against it, until one morning I woke up, realised I was wrong, apologised to the people I’d argued with, and started using ‘var’. I haven’t looked back.

It’s not that explicitly specifying type is a bad thing. It’s the need to explicitly specify type, and the need to think about type when writing a method that can be bad. When you embrace ‘var’ you tend to think about your code it a slightly different, more abstract way.

I think the same applies to Interfaces. It’s not the ‘I’ that’s the problem, it’s the need to have the ‘I’. It’s the belief that you can’t live without it. If you adopt the position that you shouldn’t care whether something is an interface or a class, it forces you to think differently about your code.

Is that change in thinking good or bad? I don’t know. I’m still exploring the question, but you only get to explore it by letting go of notions about what you need to know, and start looking critically what what value you are actually getting.

When you write a method that expects a Logger, why do you tell it to expect an ‘ILogger’?, there is no such thing. You are creating a layer of indirection in your code and in your ubiquitous language that apparently exists for no other reason than to tell you that the method accepts an interface.

By all means have the method accept objects that conform to the Logger “contract”. Define that “contract” in an interface if you want. But don’t have an ILogger and a separate class called Logger.

Logger sounds like a name for an Interface, it doesn’t sound like a great name for a class. I’m amazed by how people get upset about losing the ‘I’ in ILogger, as if it conveyed all sorts of information, but they don’t care how little information a class name like ‘Logger’ conveys. Where does this class log to? What does it log? Surely those are questions you’d like to answer at a glance more than whether or not something is an interface.

For the record, the replies were 8 votes for retaining the ‘I’ 4 for dropping it. Too small a sample size to be worth much, but still fairly decisive. That said, many of the answers were along the lines of “I’d like to drop it, but it’s such an engrained convention so, we have to keep it”. If you move the “I’d like to get rid of it. but it’s not practical” people from the keep column to the drop column the numbers become 7 to 5 in favour of dropping the ‘I’

The fact that there’s a debate about the ‘I’ at all is a good thing. It suggests that the issue of Clean Code and naming is being taken increasingly seriously. We’ve evolved from a situation where multiple lower case letters prefixing a variable name was best practice to one where a single letter prefixing a Typename is a bone of contention.

Much of what we consider necessary is really just stuff that’s familiar and comforting. If paramilitaries can go on killing each other because they can’t get their head around the fact that they don’t need to, how much easier is it for us to go on using conventions out of nothing more than habit.

When you evaluate whether practices are necessary, try to separate the attraction of familiarity, from the actual benefits and downsides that the practice offers.