+++
date = "2015-08-07T17:43:00Z"
title = "Unit Testing Events And Callbacks In C#"
abstract="How do you unit test C# events and callbacks? Here's how."
tags = ["testing", "c#", "code"]
categories = ["how to"]
banner = "img/banners/stop-wait-sign.png"
aliases = [
    "/2015/08/unit-testing-events-and-callbacks-in-c/",
]
+++

### The Problem
When you want to unit test a method it’s usually pretty simple. Call the method, pass it it’s parameters and assert against it’s return value, or some other property of the object that may have changed.

What happens when a method doesn’t return a value, or update some property? What happens when it leads (perhaps after some delay) to an event firing, or a callback getting called?

Events firing and callbacks getting called are very similar, but subtly different, so I’ll cover them both.

### A Simple Callback

Here’s the simplest scenario. You register a callback with a class when you create it, and you have a method that communicates back via that callback. Here’s our class.

{{< highlight csharp "style=tango" >}}
public class AClass {

    private Action<string> _callback;

    public AClass(Action<string> callback) { _callback = callback; }

    public void DoSomethingThatCallsBack(string str) {
      _callback(str + str);
    }
}
{{< /highlight >}}


To test this we want to test that the method passes the right result to the callback. We could use an anonymous function with an assert inside, but that test would pass even if the callback was never called.

Here’s how we do it.

{{< highlight csharp "style=tango" >}}
[Test]
public void TestCallback() {
  var actual = string.Empty;

  var aClass = new AClass((s) => { actual = s; });
  aClass.DoSomethingThatCallsBack("A");
 
  Assert.AreEqual("AA", actual);
}
{{< / highlight >}}

The anonymous function (s) => { actual = s; } has visibility of the variable ‘actual’ and so we can set it inside the callback and assert on it when we’re back in the scope of the test. This is a closure, a very common useful feature of programming with higher order functions.

### A Delayed Callback

A more useful arrangement (and more difficult to test) is a method that returns control immediately and does it’s work in the background, eventually calling back when it’s done.

{{< highlight csharp "style=tango" >}}
public async void DoSomethingThatCallsBackEventually(string str, Action<string> callback) {
  var s = await LongRunningOperation(str);      
  callback(s);
}

private Task<string> LongRunningOperation(string s) {
  return Task.Run(() => {
      Thread.Sleep(2000);
      return "Delayed" + s;
    });
}
{{< / highlight >}}

We want to assert that ‘DoSomethingThatCallsBackEventually’ returned immediately, but we also want to ensure that the callback was eventually called with the correct value. I know my long running operation takes 2 seconds so I’m going to accept the method returning in less than half a second.

{{< highlight csharp "style=tango" >}}
[Test]
public void TestEventualCallback() {
  AutoResetEvent _autoResetEvent = new AutoResetEvent(false);
  var actual = string.Empty;

  var sw = new Stopwatch();
  sw.Start();

  var aClass = new AClass();
  aClass.DoSomethingThatCallsBackEventually("A", (s) => { actual = s; _autoResetEvent.Set(); });
  sw.Stop();
 
  Assert.Less(sw.ElapsedMilliseconds, 500);
  Assert.IsTrue(_autoResetEvent.WaitOne());
  Assert.AreEqual("DelayedA", actual);
}
{{< / highlight >}}

We can assert immediately after calling the method to check that it returned quickly enough. We then need to hold off on any further asserts until the callback fires. The trick is to use AutoResetEvent. It will wait until it receives a signal to continue. We can set it inside the callback and then continue on with our asserts.

This idea was written up by [Anuraj P](https://dotnetthoughts.net/), but the original post seems to no longer exist.

### Success or Failure

What if our long running method fails? it would be nice to provide Success and Failure callbacks and have the appropriate one fire.

This is how we do it. We’ll use timeout as a way of succeeding or failing.

{{< highlight csharp "style=tango" >}}
public async void DoSomethingThatCallsbackEventuallyOrTimesOut(string str, Action<string> success, Action<string> failure, int timeout) {
  var task = LongRunningOperation(str);  
  if (await Task.WhenAny(task, Task.Delay(timeout)) == task) {
    success(await task);
  } else {
    failure("Timed Out");
  }
}
{{< / highlight >}}

We pass two callback, and a timeout duration. If ‘Task.Delay(timeout)’ completes before our long running task then we’ve lost the race and the else part of the if will fire the failure callback. If our task completes first the success callback will fire.

Andrew Arnott wrote up this elegant solution here.

We can test the success scenario like this

{{< highlight csharp "style=tango" >}}
[Test]
public void TestEventualCallbackSuccessWithTimeout() {
  AutoResetEvent _autoResetEvent = new AutoResetEvent(false);

  var actual = string.Empty;

  var aClass = new AClass();

  Action<string> onSuccess = (s) => { actual = s; _autoResetEvent.Set(); };
  Action<string> onFailure = (s) => { actual = s; _autoResetEvent.Set(); };

  aClass.DoSomethingThatCallsbackEventuallyOrTimesOut("A", onSuccess, onFailure, 2500);

  Assert.IsTrue(_autoResetEvent.WaitOne());
  Assert.AreEqual("DelayedA", actual);
}
{{< / highlight >}}

and the failure like this

{{< highlight csharp "style=tango" >}}
[Test]
public void TestEventualCallbackFailureWithTimeout() {
  AutoResetEvent _autoResetEvent = new AutoResetEvent(false);

  var actual = string.Empty;

  var aClass = new AClass();

  Action<string> onSuccess = (s) => { actual = s; _autoResetEvent.Set(); };
  Action<string> onFailure = (s) => { actual = s; _autoResetEvent.Set(); };

  aClass.DoSomethingThatCallsbackEventuallyOrTimesOut("A", onSuccess, onFailure, 1500);

  Assert.IsTrue(_autoResetEvent.WaitOne());
  Assert.AreEqual("Timed Out", actual);
}
{{< / highlight >}}

### Events

Events are very similar to Delegates, the big difference being the ability to add multiple handlers. First I’ll declare the event. It’s payload will be a string.

{{< highlight csharp "style=tango" >}}
  public class AClass {

    public event EventHandler<string> SomethingHappened;

    ...

}
{{< / highlight >}}

Just as with the callbacks we’ll start with an example that fires immediately

{{< highlight csharp "style=tango" >}}
public void DoSomethingThatFiresAnEvent(string str) {
  if (SomethingHappened != null)
    SomethingHappened(this, str + str);
}
{{< / highlight >}}

Since we can’t know if anything is watching the SomethingHappened event we have to check for null, and the EventHandler also requires the sender to be passed with the event.

Testing this event is very similar to testing the simple callback that we looked at above.

{{< highlight csharp "style=tango" >}}
[Test]
public void TestEvent() {
  var actual = string.Empty;

  var aClass = new AClass();
  aClass.SomethingHappened += (_, s) => { actual = s; };

  aClass.DoSomethingThatFiresAnEvent("A");

  Assert.AreEqual("AA", actual);
}
{{< / highlight >}}

Note our anonymous function takes two arguments because the sender object is included. We use ‘_’ to indicate we’re not interested in it.

Just like with the delayed callback, we want to be able to test events that don’t fire immediately.

Here’s an example of such a method

{{< highlight csharp "style=tango" >}}
public async void DoSomethingThatFiresAnEventEventually(string str) {
  var s = await LongRunningOperation(str);

  if (SomethingHappened != null)
    SomethingHappened(this, s);
}
{{< / highlight >}}

And here’s how we test it.

{{< highlight csharp "style=tango" >}}
[Test]
public void TestEventualEvent() {
  AutoResetEvent _autoResetEvent = new AutoResetEvent(false);
  var actual = string.Empty;

  var sw = new Stopwatch();
  sw.Start();

  var aClass = new AClass();
  aClass.SomethingHappened += (_, s) => { actual = s; _autoResetEvent.Set(); };

  aClass.DoSomethingThatFiresAnEventEventually("A");

  sw.Stop();

  Assert.Less(sw.ElapsedMilliseconds, 500);
  Assert.IsTrue(_autoResetEvent.WaitOne());
  Assert.AreEqual("DelayedA", actual);
}
{{< / highlight >}}

Here’s a slight variation on the callback that times out. Here we want to catch an event that doesn’t fire as quickly as we’d expect.

{{< highlight csharp "style=tango" >}}
[Test]
public void TestEventualEventTimesOut() {
  AutoResetEvent _autoResetEvent = new AutoResetEvent(false);
  var actual = string.Empty;

  var sw = new Stopwatch();
  sw.Start();

  var aClass = new AClass();
  aClass.SomethingHappened += (_, s) => { actual = s; _autoResetEvent.Set(); };

  aClass.DoSomethingThatFiresAnEventEventually("A");

  sw.Stop();

  Assert.Less(sw.ElapsedMilliseconds, 500);
  Assert.IsFalse(_autoResetEvent.WaitOne(1500));
  Assert.AreEqual("", actual);
}
{{< / highlight >}}