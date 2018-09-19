+++
date = "2014-04-04T00:00:00Z"
title = "Understanding The Four Rules Of Simple Design"
tags = ["design"]
categories = ["reviews"]
banner = "img/banners/4simplerules.png"
+++

I don’t normally review books, mainly because when it comes to technical books I rarely manage to read them in their entirety.

I have however just finished “Understanding the Four Rules of Simple Design” by Corey Haines and since it’s self published, and promoted pretty much exclusively through word of mouth, I thought it might be worth a short review.

The “4 Rules” mentioned in the title are the XP Simplicity Rules:

### Simple Design:

* Passes all tests
* Expresses developer intent
* Has no duplication
* Minimizes the number of classes and methods

To expand on the rules and put them in context Corey draws on 5 years of running Code Retreats in which participants work on Conway’s Game Of Life repeatedly during the course of a day, throwing away their code and starting again about every hour or so.

He deliberately avoids separating out the rules and discussing them in isolation from each other. He does this to reinforce the fact that the rules all impact on each other. Mixing them together means the book feels a little less structured than you might expect, but that doesn’t make it any less readable.

Bottom line, I bought this book, I’m glad I did. I’ve read it, and I will be reading it again.

At about 90 pages you can easily get through this book in an evening. When forewords, acknowledgements and appendices are excluded the book weighs in quite a bit less than 90 pages. Don’t skim or ignore those sections though. You may be like me and find towards the back a link to the original paper on the Law of Demeter and realise that although you’ve read lots about it you’ve never read the original source. Or you may find the link to a 1971 paper on designing for maintainability.

These links to other sources are one reason why the book is so short. It doesn’t cover the same well trodden ground that has been covered adequately elsewhere. The law of demeter is mentioned, but little more than that. The SOLID principles likewise are mentioned, but only to demonstrate that following the four rules tends to deliver SOLID designs.

As I started reading I got a little concerned, the examples of tests and code seemed to be fleshing out a domain model rather than tackling any particular behaviour. I’m no expert on Simple Design but this isn’t the approach I’d take. A few pages later however Corey addresses that very concern and shows how the alternative approach of tracing tests and code back to required behaviour can give better results than state based test driving of models.

Interestingly the book doesn’t implement the Game of Life in full, and doesn’t show how some of the more naive/complex implementations can be greatly simplified. I think that would have been fascinating, but I understand why it wasn’t done. Early in the book Corey is at pains to point out that he wants to avoid focus on “A Good Design” or “The Best Design”. If he had provided anything like a full implementation, it would have been hard to avoid the impression that after 5 years of Code Retreat here was (in his view) “The best design”.

That said, I think the book might have benefited from some additional examples and more detailed discussion. It seemed to end just when things were getting really interesting. The section on inverted composition as an alternative to inheritance threw up a whole host of questions in my mind. Corey concluded that discussion quite quickly with an “I’m not saying this is or is not better” kind of non-conclusion that felt a little unsatisfactory.

If you are the kind of person who reads about and thinks about software design on a regular basis, you may be tempted to write this book off as covering ground you’re familiar with. I wouldn’t do that. For the small investment of time and money that the book requires it does contain a few Aha! moments.

If you don’t read about software design very much then this book is actually a really good place to start. There is nothing complicated or advanced here, There is nothing here that can’t be put into practice immediately.

If you really haven’t thought about code as design then you WILL be a better developer for having read this book. Maybe not massively better. Nothing that can be read in an evening will make a dramatic difference, there are no short cuts. But, If you are looking at the elephant of mastering software design, and trying to figure out how to eat it, this is a small enough bite to get you started.

I suspect this book might benefit from multiple readings, and I’m sure that to get the most from it you really should follow Corey’s advice and read the additional material that he links to.

One final thought, although the book on it’s face seems to discuss Object Oriented design, it really isn’t written in an OO specific way. At the moment most of my spare energy is going into understanding how a functional approach can simplify design. I didn’t feel like I was parking that to “go back” to OO thinking. Everything in the book can be applied to Functional Programming. In fact, reading this book through a functional lens actually throws up a lot of extra questions.

Bottom line, I bought this book, I’m glad I did. I’ve read it, and I will be reading it again.