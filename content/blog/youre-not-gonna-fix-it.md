+++
date = "2012-01-17T02:00:00Z"
title = "You're Not Gonna Fix It"
tags = ["technical debt", "coding"]
categories = ["thoughts"]
banner = "img/banners/wrench.png"
+++

I’m not going to justify kludges, or apologise for kludges. I don’t need help figuring out how to avoid them. Kludges don’t usually come about because we don’t know how to avoid them. They usually exist because we make a judgement call. We decide that a kludge is not worth avoiding. Dress it up any way you like, but it comes down to a decision.

This post starts from the premise that in all liklihood there will always be kludges. I want to talk about the lie that programmers tell ourselves every time we resort to a kludge.

“I’ll fix that later”.

We may include a comment indicating that we’ve done something we’re not proud of. We may provide details about how to fix it.

We could start the comment with a word like HACK, so we can find these gems later.

Let’s knock something on the head right now.

You’re Not Gonna Fix It.

If that rings a bell it has echoes of “You’re not gonna need it”.

“You’re not gonna need it” serves as a warning against doing too much in the hopes that it will be useful later. 

“You’re not gonna fix it” warns us against doing too little in the hopes we can fix it later.

Let’s assume you’ve hit one of those moments where there’s a 5 minute fix.

Let’s assume that you’re unhappy with the quick solution. There is a better way. For the purposes of this post it doesn’t matter what the better way is.

You make the judgement call to go with the 5 minute fix. You do the fix, release the code, it gets through QA, pushed into production, and it works. Great.

A month later, you find yourself with a spare afternoon. A chance to crack open the code and take a swing at some of those kludges.

Now, depending on your development process this may actually be a dumb idea. If production changes need to go through QA then you are about to create a chunk of work for the QA team. Have you checked with them that they have capacity?

What if this fails right as you get started on your new project, will you have capacity?

We’re talking about a change that brings no visible value to the users. Are you going to run it by your manager?

Should a manager ok a production change to tidy up some code that was gnawing at a developers conscience?

If you claim that the fix is trivial, it begs the question, why wasn’t it done right in the first place. 

In reality changes only get harder with the passage of time. You are never going to be as familiar with the code as you were when you first created the kludge. No amount of HACK comments can fix that.

The more “Waterfallish” your process the less likely it is that you’ll be able to fix issues later. The more people involved in a release, the more likely it is that issues will remain untouched.

So for some, "You’re not gonna fix it" is just a harsh reality. It’s how it should be. There’s a cost associated with being able to fix kludges. Either reduce the friction associated with releases, or get used to the fact that a kludge is for life.