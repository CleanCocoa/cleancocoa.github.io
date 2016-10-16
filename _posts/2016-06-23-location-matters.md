---
title: Location Matters
created_at: 2016-06-23 09:07:27 +0200
kind: worklog
tags: [ refactoring, srp ]
comments: on
---

The location of a piece of code always matters. I was fiddling with the [Charts][] library today and found out that if you set a chart's data to `nil`, it renders some informative text by default. My data source doesn't pass `nil` but an empty array around, though, so I had to convert empty arrays to `nil` to make use of this feature.

Take this simple code:

    #!swift
    func drawLineGraph(dataPoints: [DateBasedDataPoint]) {

        guard !dataPoints.isEmpty else {
            chartView.data = nil
            return
        }

        chartView.data = lineChartData(dataPoints: dataPoints)
    }
    
    private func lineChartData(dataPoints dataPoints: [DateBasedDataPoint]) 
        -> LineChartData {
        /* ... */
    }

That's my API's entry point. The logic is simple enough and it achieves the desired effect. Either set `data` to `nil` or transform the values.

Now take this other version after a simple refactoring:

    #!swift
    func drawLineGraph(dataPoints: [DateBasedDataPoint]) {

        chartView.data = lineChartData(dataPoints: dataPoints)
    }
    
    private func lineChartData(dataPoints dataPoints: [DateBasedDataPoint]) 
        -> LineChartData? {
            
        guard !dataPoints.isEmpty else { return nil }
        
        /* ... */
    }

There, the conversion function takes care of returning `nil` if the `dataPoints` array is empty. 

What's the gain?

The `drawLineGraph` method now _only_ takes care of obtaining compatible values and assigning them to the `chartView`. It doesn't know what else to do with the `dataPoints`. Before, the method had knowledge of doing 2 things: obtaining values or setting the chartView to an "empty" state, depending on knowledge about the `dataPoints`.

Even though the `guard` statement is quite similar, moving the decision to a place where intimate knowledge about creating chart view data is required makes more sense and simplifies the call site (`drawLineGraph`).

When I'm done fiddling around with this, I can promote the conversion from a private method into an object on its own. Then the converter will have all the knowledge required to create data that the Charts library understands.

[charts]: https://github.com/danielgindi/Charts
