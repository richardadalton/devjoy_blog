+++
date = "2014-07-09T17:43:00Z"
title = "Fly The Damn Plane"
tags = ["agile"]
categories = ["thoughts"]
banner = "img/banners/plane.png"
+++

![Car](/img/plane.png)

### Constant Learning
Being a software developer means constant learning. The technical landscape is always shifting. We have to run to stand still. We know this. We accept it. For some it’s the very thing that attracts them to the profession.

I’ve learned lots about software development in the last few years.

How to automate builds
How to automate tests
Object Oriented Programming/Design
Functional Programming/Design
Operating Systems
Programming Languages
Frameworks
Version Contol Systems
I’ve tried to embrace Agile, hell I’m even a certified Scrum Master. I attend conferences, speak at conferences, read lots and blog a little.

Despite all this I feel I am a worse “Software Developer” than I used to be. Which can be partly explained by this quote from John Archibald Wheeler.

“We live on an island surrounded by a sea of ignorance. As our island of knowledge grows, so does the shore of our ignorance.”

In other words, the more you learn, the more stupid you feel.

This, combined with the Dunning-Kruger effect suggests that feeling like we’re getting worse, even as we get better might be understandable.

But that’s not it. It would be great to explain this all away, pretend it’s all in the mind, but I don’t think it is. I believe I am actually a worse developer now than I used to be. Less productive, less focused, less comfortable.

And, I think I know why.

### Fly The Plane
In an emergency pilots are trained to remember that their first priority is to “fly the plane”. It may seem odd that they need to be reminded of that fact, but it’s incredibly easy to become focused on an instrument that doesn’t work and forget to keep the plane in the air. On December 29th 1972 Eastern Air Lines Flight 401 crashed into the Florida Everglades with 101 fatalities. The flight crew were all focused on a burned out landing gear bulb and failed to notice that the autopilot wasn’t maintaining altitude.

They weren’t bad pilots, or bad people, they made a mistake, a mistake that we all make, all the time. The consequences of focusing on an immediate issue, forgetting to fly the plane, trusting that the autopilot had their back was catastrophic. They paid the ultimate price.

The consequences to the rest of us of making a similar mistake are far more benign, but there are consequences. I don’t think I’m alone in letting a focus on “building software the right way” distract from the real job of “building software”.

For me, it all started to go wrong when I started learning TDD.

Devouring everything I could read about TDD gave me a glimpse of an Agile world, of continuous integration, continuous deployment, executable specifications, distributed version control systems and feature branches. A world where the software development process worked. A magical place where you could pick requirements off the trees and working software flowed like a mighty stream.

Knowing that such a magical place existed became a curse. It made me resent the daily frustrations that have always blighted software developers. It made me feel bad every time I wrote code that didn’t have tests. It made me obsess over design and clean code to the point that sometimes I froze unable to move forward, It made me waste hours trying to automate things that absolutely needed to be automated, but not at the cost of shipping software.

### Tools Tools Tools
The Agile manifesto proposes “Individuals and interactions over processes and tools”. And yet, processes and tools are deployed in ever growing numbers in an attempt to “be agile”. I’ve spent a huge chunk of my time on tools like Team City, Jenkins, Git, Subversion, Testrail, Rally, Jira, FitNesse, RSpec, NUnit, Vagrant, Puppet, VirtualBox, and all that before we even get to a programming language.

You can study all of those tools, learn about 20% of each of them and still not know a damn thing about delivering software other than it’s really hard to get tools to talk to each other.

### A Call to Action
Here’s what I’ve started doing, and if my sorry tale sounds familiar you might like to join me.

Stop.

Go and build a product. Any product, but make it a complete product, it can be small but it must actually do something. I’ve started with a tool to rank the pool players in our office using Elo Ratings.

Don’t use any tools other than your IDE.

Open up Visual Studio or RubyMine or whatever and build a product.

Don’t write unit tests, don’t automate the build or the deployment. Don’t try to be agile.

When you have a fully working product, build another, and another, or built the same product again.

Forget the Red-Green-Refactor Rhythm, get into the rhythm of building working products, not functions, not programs, “Products”. Don’t just throw any old crap up and call it a product, apply a little polish. Pretend you are delivering for a client. Start by delivering the core value of the product and then improve it.

Do as much as possible manually so that you get your mind back to the bare bones of building software. What do you actually need to do?

Get faster at delivering. You should be able to build a small app in a few hours. Build the same app multiple times, Katas don’t have to be about Test Driven Development of tiny functions. Do a “Ship A Product” Kata, build a product in an hour, by hand, then throw it away and build it again.

Once you’ve got that rhythm going then, AND ONLY THEN, add in an automated build. When you’ve got that working then AND ONLY THEN add in automated tests.

For my first stab at this I didn’t even use version control, I put the code in dropbox.

Don’t do anything because “It’s the right thing to do, or it’s agile”, only do things because you can see that it makes sense, makes you faster, makes life easier, solves an ACTUAL problem.

### Minimum Viable Product
Working Software is the minimum viable product from your software development process. An automated build that delivers nothing is just wasted time. You can spend a long time creating an all encompassing Walking Skeleton and never start on the product.

Focus on actually completing some small products. Figure out what you really need. Evolve your Walking Skeleton from first principles.

This isn’t an anti-agile or anti-tdd post. Quite the opposite. We need to take an agile approach to being agile. Working software is our green light, that’s the baseline. If adopting any agile practice hinders your ability to deliver working software then revert, get back to green and try again, or try something else.