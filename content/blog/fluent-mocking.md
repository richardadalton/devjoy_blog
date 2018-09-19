+++
date = "2011-06-22T17:43:00Z"
title = "Fluent Mocking"
tags = ["testing", "c#"]
categories = ["how to"]
banner = "img/banners/facade.png"
+++

Here’s a scenario (for once not a hypothetical scenario, this is a real system I worked on). I was building a wizard based app. To be more accurate I was building lots of wizard based apps.

After a couple of wizards the functionality became clear and I extracted it to it's own framework. My apps could then focus on actual bread and butter functionality. 

Two of the objects in my framework are ‘Wizard’ and ‘WizardStep’. A Wizard contains WizardSteps. You get the idea.

### Testing Our Framework
There are quite a few tests that we can write to make sure that navigation works. 

* Moving next should increment the current step.
* Moving previous should decrement the current step. 
* Moving next from the last step causes the Wizard to finish. 
* You can't move previous from the first step.
* A step can enforce a check to decide whether to allow a move.

Let’s look at we can write these tests, and how we can use mocks and stubs. 

When the user clicks ‘Move Next’ the current step can decide whether it will allow the Move Next to happen. If it returns false, the Wizard will refuse to allow the user to move on.

To test a feature like this we can do the following:

1. Create a wizard with one step
2. Use a stub for step1 that returns false from OKToMoveNext.
3. Start the wizard
4. Assert that we’re on the first step
5. Attempt to move next
6. Assert that we’re still on the first step

We should still be on step1 because step1 returns false when asked if it's ok to move next.

We can write the test in various ways. A key issue is how to write the stub for step1. The only thing we care about is that it's OKToMoveNext method returns False. 

Here’s one example, using the Moq Framework:

{{< highlight csharp "style=tango,hl_lines=4-5" >}}
[Test()]
public void Validation_CanPrevent_MoveNext()
{
    Mock<IWizardStep> step1 = new Mock<IWizardStep>();
    step1.Setup(s => s.OKToMoveNext()).Returns(false);

    Wizard wizard = new Wizard()
        .AddStep(step1.Object)
        .Start();

    Assert.AreEqual(step1.Object, wizard.CurrentStep);
    wizard.MoveNext();
    Assert.AreEqual(step1.Object, wizard.CurrentStep);
}
{{< / highlight >}}

I don’t like this code. It’s too busy. There’s too much “stuff” that’s related to the mocking framework. The intent of the test might be discernible, but only just. The shaded lines in particular need a second or third glance to make sure you’re reading them right. Our intent is to create a stub wizard step that can’t move next. Our test should be screaming that intent so clearly that it cant be missed by someone reading the code.

Scenarios like this may be why [some developers](https://serialseb.com/blog/2016/09/13/what-is-so-wrong-with-mocking-libs/) dislike mocking frameworks. The same test using hand-coded classes is much simpler and its intent is clearer: 

{{< highlight csharp "style=tango,hl_lines=4" >}}
    [Test()]
    public void HM_Validation_CanPrevent_MoveNext()
    {
        IWizardStep step1 = new WizardStepThatIsNotOKToMoveNext();
 
        Wizard wizard = new Wizard()
                                .AddStep(step1)
                                .Start();
 
        Assert.That(wizard.CurrentStep == step1);
 
        wizard.MoveNext();
 
        Assert.That(wizard.CurrentStep == step1);
    }
{{< / highlight >}}

The shaded code tells most of the story. Because we’re creating a simple class for a specific purpose, we can be very explicit with our naming.

Although listing 2 is an improvement over the code we produced using the Moq mocking framework, it’s not without problems of its own.

Our suite of tests is going to need a lot of different mocked WizardSteps to cover the various scenarios. Many will be very similar, or will have parts that are identical to parts of others. 

We could have a dozen versions of the class that need to prevent a user Moving Next, but each may need to do that in conjuction with some other different behaviour.

We could try to make our Handmade mocks more intelligent, but that’s a slippery slope. Adding logic into our hand rolled mocks leads to questions about to test our mocks.

One interesting option is to go back to using our Mocking Framework, but hide the messiness of it behind a nicer abstraction. Imagine being able to write a test like the one in Listing 3:

{{< highlight csharp "style=tango,hl_lines=4-6" >}}
    [Test()]
    public void step_can_stop_move_next()
    {
        IWizardStep step1 = new MockWizardStep()
                                .ThatCannot.MoveNext
                                .Object();
 
        Wizard wizard = new Wizard()
                                .AddStep(step1)
                                .Start();
 
        Assert.AreEqual(step1, wizard.CurrentStep);
 
        wizard.MoveNext();
 
        Assert.AreEqual(step1, wizard.CurrentStep);
    }
{{< / highlight >}}

This is a fluent style interface, but behind the scenes it’s doing all the same stuff that our first test did. 

Once you’ve written the Factory, you can use it to spit out other variations of the mocked object.

{{< highlight csharp "style=tango" >}}
        IWizardStep step1 = new MockWizardStep()
                                .ThatCan.MoveNext
                                .Object();
 
        IWizardStep step1 = new MockWizardStep()
                                .ThatCan.MoveNext
                                .ThatCannot.MovePrevious
                                .Object();
{{< / highlight >}}

The fluent interface in place it gets a lot easier to create exactly the right mock for the scenario you want to test. 

The test becomes clearer, and to a certain extent you’ve abstracted your tests away from the specific mocking framework that you are using.

Of course, you have to actually build the fluent interface. A fluent interface by it’s nature is a Domain Specific Language. You base the language on the properties of the objects you’ll be mocking.

Creating the Fluent Interface isn’t a particularly complicated task, but there's a knack to it. 

Look again at the shaded code in Listing 5. It appears to be a readonly property, modifying fields within the class. What madness is this?

This is a trick used in fluent interfaces to avoid parenthesis after every term in a statement.

{{< highlight csharp "style=tango,hl_lines=11-18" >}}
public class MockWizardStep
{
    private Mock<IWizardStep> _step;
    private bool _thatCan = true;
 
    public MockWizardStep()
    {
        _step = new Mock<IWizardStep>();
    }
 
    public MockWizardStep ThatCan
    {
        get
        {
            _thatCan = true;
            return this;
        }
    }
 
    public MockWizardStep ThatCannot
    {
        get
        {
            _thatCan = false;
            return this;
        }
    }
 
 
    public MockWizardStep MoveNext
    {
        get
        {
            _step.Setup(v => v.OKToMoveNext()).Returns(_thatCan);
            return this;
        }
    }
 
    public MockWizardStep MovePrevious
    {
        get
        {
            _step.Setup(v => v.OKToMovePrevious()).Returns(_thatCan);
            return this;
        }
    }
 
    public IWizardStep Object()
    {
        return _step.Object;
    }
 
    public Mock<IWizardStep> Mock()
    {
        return _step;
    }
}
{{< / highlight >}}

So, where does that leave us? Are mocking frameworks saved from those who hate them?

Perhaps not, here’s one way to abstract your tests from your mocking framework, and maybe write clearer tests.