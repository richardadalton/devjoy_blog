+++
date = "2013-10-23T12:00:00Z"
title = "Don't Scatter Logic All Over The Call Stack"
tags = ["code"]
categories = ["how to"]
banner = "img/banners/keep-tidy-sign.png"
+++

Here’s a problem I come across all the time in legacy code and I have been guilty of it myself in the past. This isn’t going to be rocket science, but, apparently, this stuff doesn’t go without saying. Normal caveats about this being a simplified contrived example apply.

Take a look at this code

{{< highlight fsharp "style=tango" >}}
static void Main(string[] args)
{
    PrintTradeBalance();
    Console.ReadKey();
}
{{< /highlight >}}

Oh! to have code this simple right? It’s pretty clear what it does, it prints a trade balance, whatever that is. Let’s not break out the bunting and brass band yet. What does this code ACTUALLY do. Let’s dig into the function.

{{< highlight fsharp "style=tango" >}}
private static void PrintTradeBalance()
{
    var results = GetTradeBalance();

    foreach (var result in results)
        Console.Write("Country {0}, Value {1}\n", result.Country, result.Value);
}
{{< /highlight >}}

Well it turns out a TradeBalance isn’t a thing it’s a collection, each country has one. Let’s leave the function name aside for a moment. We see the function GETS the trade balances, then prints them. It’s still not really clear what a trade balance is or where it comes from. Lets’ keep digging.

{{< highlight fsharp "style=tango" >}}
private static IEnumerable<Result> GetTradeBalance()
{
    var cashMovements = GetCashMovements();

    var results = cashMovements
        .GroupBy(i => i.Country)
        .Select(r => new Result
        {
            Country = r.Key,
            Value = r.Sum(c => c.ValueIn - c.ValueOut)
        });

    return results;
}
{{< /highlight >}}

Now we’re getting somewhere. We get CashMovements and aggregate them by Country, Netting the ValueIn against the ValueOut. But what are CashMovements? Where do they come from?

{{< highlight fsharp "style=tango" >}}
private static IEnumerable<CashMovement> GetCashMovements()
{
    var transactions = GetTransactions();

    var movements = transactions
        .GroupBy(i=>new { i.Country, i.Direction })
        .Select (m=>new CashMovement
        {
            Country = m.Key.Country,
            ValueIn = m.Where(x => x.Direction == "Export").Sum(c => c.Price * c.Quantity),
            ValueOut = m.Where(x => x.Direction == "Import").Sum(c => c.Price * c.Quantity)
        });

    return movements;
}
{{< /highlight >}}

CashMovements are an aggregation of Transactions, which have a direction (Import/Export) a price and a Quantity.

We could go further and see where the transactions come from, but lets assume they are pulled from a DB.

### Problems
Oh code! how do I hate thee? Let me count the ways.

We had to burrow down three levels deep just to see where the source data came from.
The transformation of the data was spread over those functions we drilled into.
We encountered the steps of the transformation in reverse order as we drilled down.
Does any of this matter?

Yes. I think it does.

First let’s address one concern you might have. Yes, the aggregation that’s going on here is relatively simple and could probably be done in one function. That’s not really the point. In practice these kinds of transformations, aggregations and enrichment are far more complicated and this drilling down deeper and deeper into functions calls is not uncommon.

Having to burrow down into a function should mean we are going into more detail, a lower lever of abstraction. In this case that’s not really what’s happening, we are simply drilling down into the NEXT step of the overall algorithm. All of these steps are essentially at the same level of abstraction.

We lose any sense of the overall transformation by stringing together function calls like this. Worse, as mentioned above, the algorithm is actually upside down. We drill down 3 levels to get the source data and then transform it on the way back up the call stack. We could hardly find a less intuitive way of describing the algorithm if we tried.

We’ve also coupled each function to the next. Each of these functions does something useful to data, but we don’t provide the data, The functions pull it out of the next function, which means they need to be aware of the next step in the chain. This is a fundamentally flawed way of thinking about functions, it makes life needlessly complicated. The code is harder to test, harder to understand, harder to maintain.

So, why is this kind of code so common?

I believe it may be a problem in the way we teach programming. We tell students to decompose problems from the top down, pushing complexity down into lower level functions. The result is code like that described above.

There are two kinds of complexity and I don’t think students of programming are adequately taught to understand and handle the different kinds.

The first kind of complexity is domain complexity, the problem at hand. The seemingly arbitrary business rules and edge cases that seem to keep popping up. This kind of complexity is vitally important. This is the stuff your client will ask you to explain, ask you to change, ask you to validate. This sort of complexity shouldn’t be pushed down, it should be pulled out, highlighted.

The transformation of transactions into Trade Balances is a Use Case of our system. The logic needs to be easily accessible and future developers need to be able to grasp what’s happening quickly and change it with confidence.

The other kind of complexity is implementation details. You need to go through a Web Service to get the transactions? That’s an implementation issue. You pull the data from a Database and map it to structs or classes? that’s an implementation issue. These sorts of complexity should be pushed down, hidden, abstracted away so that the business logic stands out.

In the code above we shouldn’t be pushing logic down to lower and lower functions. Spreading the algorithm over the call stack by “pushing down” is nuts. We should be pulling the algorithm up, embracing it, making it jump off the screen when someone opens our code.

This isn’t a rant about functional programming, it doesn’t matter whether you use functional, OO or procedural code the principle here is the same.

The following modified code isn’t astonishing or revolutionary or even beautiful, but it’s better, a little better, and if we could all just get a little better life would be so much easier, legacy code would be a much smaller problem. The changes aren’t even difficult. This is just a question of internalising a very simple idea and you’ll never write code like the code above again.

{{< highlight fsharp "style=tango" >}}
static void Main(string[] args)
{
    var transactions = GetTransactions();
    var cashMovements = GetCashMovements(transactions);
    var tradeBalances = GetTradeBalances(cashMovements);
    PrintTradeBalances(tradeBalances);
    Console.ReadKey();
}
{{< /highlight >}}

We get Transactions, turn them into CashMovements, and turn those into TradeBalances and print them.
We can now drill down meaningfully into any of those steps to see how each is done. That is valid use
of drill down, we are going to a lower level of abstraction.

The algorithm also reads the right way around. We start with Transactions and end with TradeBalances.

The only change to the functions is that instead of each calling the next function in the change, each is passed the data that it operates on as a parameter.

{{< highlight fsharp "style=tango" >}}
private static IEnumerable<CashMovement> GetCashMovements(IEnumerable<Transaction> transactions)
{
    var movements = transactions
        .GroupBy(i => new { i.Country, i.Direction })
        .Select(m => new CashMovement
        {
            Country = m.Key.Country,
            ValueIn = m.Where(x => x.Direction == "Export").Sum(c => c.Price * c.Quantity),
            ValueOut = m.Where(x => x.Direction == "Import").Sum(c => c.Price * c.Quantity)
        });

        return movements;
}

private static IEnumerable<TradeBalance> GetTradeBalances(IEnumerable<CashMovement> cashMovements)
{
    var tradeBalances = cashMovements
        .GroupBy(i => i.Country)
        .Select(r => new TradeBalance
        {
            Country = r.Key,
            Value = r.Sum(c => c.ValueIn - c.ValueOut)  
        });

        return tradeBalances;
}

private static void PrintTradeBalances(IEnumerable<TradeBalance> tradeBalances)
{
    foreach (var tradeBalance in tradeBalances)
        Console.Write("Country {0}, Value {1}\n", tradeBalance.Country, tradeBalance.Value);
}
{{< /highlight >}}
