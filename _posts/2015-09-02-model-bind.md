---
title: Where Instead of Using Functional Bind, I Create an Expressive Model
created_at: 2015-09-02 15:19:45 +0200
kind: worklog
tags: [ functional-programming, ddd, model, builder, design-pattern, oop ]
shared_image: blog/201509021531_clean-up-t.jpg
vgwort: http://vg08.met.vgwort.de/na/e70924d9c849444d8d825ab7b2372cb0
comments: on
---


[The other day][post], I wrote a post about `bind()` and the `>>=` operator and how it can help chain function calls together. The example was a bit too contrived and making it fit the requirements left us with really bad code. I came up with an even better implementation: use plain Swift objects and express your intent carefully.

Let's have a look what we left of with last time.

In order to chain functions with `bind()`, the incoming parameters of one function define a contract which the function before has to satisfy through its return value.

The end result looked quite slick:

    #!swift
    withdrawCash(amount, fromAccount) 
      >>= depositCash(toAccount)
      >>= addTransferToLog

But Recall what the final functions looked like for this to work, put into an order o you can see the dependencies:

    #!swift
    func addTransferToLog(amount: Money, from: Account, to: Account) { ... }
    func depositCash(to: Account)(amount: Money, from: Account) 
        -> (Money, Account, Account)? { ... }
    func withdrawCash(amount: Money, from: Account) 
        -> (Money, Account)? { ... }

Returning a tuple is possible in Swift, but it's far from elegant.

There are is an alternate versions which I like better. The good old Builder. 

## Using a Builder, Uncovering API Mistakes

One of the original [*Gang of Four* Design Patterns][gof].[^aff] Using a Builder object, the result can become:

    #!swift
    if let transfer = TransferBuilder()
      .sendMoney(amount)
      .from(fromAccount)
      .to(toAccount)
      .build() { ... }

The `build` method can obviously fail when not all parameters off a transfer are met.

For so simple a data structure, an initializer will work just as well.

    #!swift
    struct Transfer {
        init(amount: Money, fromAccount: Account, toAccount: Account) { ... }
    }

Worse, though, this model unveils a mistake in the original code. There, we chained two events for the sake of showing what bind can do: withdrawing and depositing cash.

I found I cheated when I made `depositCash` require a `fromAccount` parameter just to pass it along. Cash depositing does not require such things. Money transfers do.

## Using no Builder but Expressive Objects

{% include figure.md src="/assets/blog/blog/201509021531_clean-up.jpg" alt="Please Clean Up Your Mess sign" caption="Photo Credit: <a href=\"https://www.flickr.com/photos/allen_goldblatt/194069411/\">Please Clean Up Your Mess</a> by Allen Goldblatt. License: <a href=\"https://creativecommons.org/licenses/by/2.0/\">CC-BY 2.0</a>" %}

So while the Builder and the value type initializer now better capture my real intent, I see that I miss cash withdrawal and depositing. The behavior is missing.

The Builder design pattern and `bind()` work well to create and transform data, respectively. They don't express behavior, though. That's something you don't learn from patterns. But Domain-Driven Design has taught me well, so here's a different approach to cash withdrawal.

Turning to explicit modeling practices, cash withdrawal becomes a process with quite a few actors involved:

    #!swift
    extension Account {
        func withdrawCash(amount: Money, 
            usingCashMachine cashMachine: CashMachine,
            clock: Clock) -> Result<Payout, InsufficientFunds> {
        
            if !amountWithdrawalIsWithinAllowance(amount) { 
                return .Failure(InsufficientFunds(account: self, 
                    balance: currentBalance))
            }
            
            let payout = Payout(account: self, amount: amount, time: clock.now())
            cashMachine.performPayout(payout) // infrastructure: shell out real cash
    
            return .Success(payout)
        }
    }

The original functions looked cool. But I picked a bad example: one where sensitive information is involved and transactions have important invariants. Modeling a real object is more versatile and closer to the problem domain.

My takeaway is this: don't "functionalize" just everything only because the pattern is cool. Look for really good use cases for bind (like data transformations) and stick to explicit and intent-revealing objects for the main part of the work.


[gof]: http://amzn.to/1UrtD6G
[post]: /posts/2015/08/east-bind/

[^aff]: Affiliate links; I get a small kickback from the vendor if you buy from my link but it won't cost you anything.

---

Photo Credit: [Please Clean Up Your Mess](https://www.flickr.com/photos/allen_goldblatt/194069411) by [Allen Goldblatt](https://www.flickr.com/photos/allen_goldblatt/). License: [CC-BY-2.0](https://creativecommons.org/licenses/by/2.0/).
