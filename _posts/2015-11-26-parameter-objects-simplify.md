---
title: Parameter Objects Simplify Your API Versions
created_at: 2015-11-26 13:08:16 +0100
kind: worklog
tags: [ refactoring, api, versioning ]
comments: on
---

I watched an AltConf about API design the other day. I cannot seem to find which talk it was, though. Anyware, the presenter talked about using parameter objects when the parameter list grows too long or is open for change in future versions.

Parameter objects can change internally and evolve with the API. You can add or remove attributes, for example, while the API calls of old client code don't have to change: they still pass the same type in. That makes framework updates a bit less painful because method signatures stay the same. 

Instead of:

    #!swift
    func banana(length: Double, weight: Double, origin: String, color: UIColor)

You use:

    #!swift
    struct BananaData {
        let length: Double
        let weight: Double
        let origin: String
        let color: UIColor
    }
    
    func banana(data: BananaData)

You have to provide sensible defaults for new attributes, given they're not required upon initialization. If so, client code has to update its factories of course, which waters down the benefits of parameter objects. Still this can be better than changing method signatures: if parameter objects are reused on the client side, client code will change less.

Instead of default values, it's totally possible to make new parameters optional and thus defer the work to the framework instead of the client.

So if before your API was:

    #!swift
    func sendMoney(amount: Money, from: Account, to: Account)
    func logTransaction(amount: Money, from: Account, to: Account? = nil)

After it could look like that:

    #!swift
    struct Transfer {
        let amount: Money
        let sender: Account
        let receiver: Account?
    }
    
    func transferMoney(transfer: Transfer)
    func logTransaction(transfer: Transfer)

What if the new `transferMoney` receives a transfer object without receiver? It withdraws cash, maybe, or aborts the process. (Okay, maybe this is just not the very best example :))

Parameter objects are great to stay sane when method signatures are volative and to group related values together under a meaningful title. [Introduce Parameter Object][par] is a refactoring in [Fowler's classic book][ref], by the way: put together into a new value object what goes together.


[par]: http://www.refactoring.com/catalog/introduceParameterObject.html
[ref]: http://amzn.to/1PQj4fV
