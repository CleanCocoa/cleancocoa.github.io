---
title: Refactoring to Clean Value Transformations in Swift
created_at: 2016-04-09 10:08:48 +0200
kind: worklog
tags: [ clean, refactoring ]
url: https://realm.io/news/doios-daniel-steinberg-ready-for-the-future/
comments: on
---

Daniel Steinberg's presentation ["Ready for the Future: Writing Better Swift"][swift] teaches us a lot about readable code. He refactors a calculation into many functions with very specific responsibilities. The resulting functions are super slim.

The result is this:

    #!swift
    let lastWeeksRevenues = lastWeeksSales
                            » anArrayOfDailySales
                            » replaceNagiveSalesWithZeroSales
                            » calculateRevenuesFromSales

When you read that code it should be very clear what's going on. The function names communicate their intent.

Now the implementation:

    #!swift
    func anArrayOfDailySales(rawSales: AppSales) -> [Int] {
        return rawSales.map{$0}
    }
    
    func replaceNagiveSalesWithZeroSales(sales: [Int]) -> [Int] {
        return sales.map(negativeNumbersToZero)
    }

    func negativeNumbersToZero(number: Int) -> Int {
        return max(0, number)
    }
        
    func calculateRevenuesFromSales(sales: [Int]) -> [USDollars] {
        return sales.map(revenuesInDollarsForCopiesSold)
    }

    func revenuesInDollarsForCopiesSold(numberOfCopies: Int) -> USDollars {
        return numberOfCopies
            » revenuesForCopiesSold
            » toTheNearestPenny
    }
    
    let unitPrice = 1.99
    let sellersPercentage = 0.70
    
    func revenuesForCopiesSold(numberOfCopies: Int) -> USDollars {
        return Double(numberOfCopies) * unitPrice * sellersPercentage
    }
    
    infix operator » {associativity left}

    func »<T, U>(input: T, transform: T -> U) -> U {
        return transform(input)         
    }
    
Apart from the `»` operator which merely changes `foo.map(bar)` to `foo » bar`, the functions are very small, easy to read. This reminds me of the result of 3/4 of Sandi Metz's [arbitrary rules](https://robots.thoughtbot.com/sandi-metz-rules-for-developers) for writing Ruby code:

1. Classes must be shorter than 100 lines
2. Methods must be shorter than 5 lines
3. Always pass less than 4 parameters into a method

Of course the result is a lot of functions for a rather simple algorithm. But it works well because it is easy to read _in the long term_. Even newcomers can grasp what's going on without knowing much about the language or typical Cocoa-programmer conventions. 

Bonus: you can unit test each function to check if the parts of the overall algorithm works as expected.

[swift]: https://realm.io/news/doios-daniel-steinberg-ready-for-the-future/
