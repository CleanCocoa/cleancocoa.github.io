---
title: "My Declarative Breakthrough: Wherein I Stop Thinking in Terms of Object Collaboration"
created_at: 2016-10-12 11:14:29 +0200
kind: worklog
tags: [ frp, declarative ]
vgwort: http://vg01.met.vgwort.de/na/de0ae83a1c534331872748b37e3d36fb
comments: on
---

I just grokked how value transformations can _not_ be a pain in tests. I'm writing a simple export module which takes a table from TableFlip and renders it as LaTeX. So the input is a `Table` and the output a `String`. 

But the module is not just a single function, mind you. That'd be a huge function, and not very open for configuration changes later. I factored it into a few sub-components, of course.

Now how do I write tests for the object which does convert a table to a string when I also have to write tests for the sub-components without merely duplicating the test cases and without writing integration tests?

That's when it clicked.

## How I First Ended up With an Integration-Test Situation

The conversion is made up of these components:

* `LatexRenderer` -- the module's public interface
* `LatexExportBuilder` -- used to configure rendering
* `LatexTableBody` -- converts tabular data to strings with "&" etc.

The `LatexExportBuilder` accepts 5 different settings and its `build()` method uses a LaTeX template to render the actual string. One of its non-optional settings is the LaTeX table body, a string itself. The builder only puts the values into proper places in the template; that's all it does. That's enough of a responsibility already.

From there follows that rendering the table body is the responsibility of another object. I call it `LatexTableBody`; it doesn't _do_ anything, it _is_ the body. You put in a table upon initialization and it exposes methods to access the string rendition (or serialization) of the table. In Java-inspired object-oriented design, this would be a `LatexTableBodyRenderer` with a `render()` method. These are action based names. I chose data-based names without a notion of activity; its methods are called `bodyRows() -> [String]` and `headerRow() -> String?`. For convenience, it also exposes `joinedRows() -> String`. You see, no activity, just state transformations.

Then there's a class which sets everything up. Its the `LatexRenderer`. It configures the builder and has a method `render() -> String` which delegates to the other objects for conversion.

I wrote `LatexTableBody` tests already. And I wrote `LatexExportBuilder` tests for template configuration. The renderer uses them both. How should I write assertions for the renderer when its behavior is basically the product of both existing test suites?

## Changing the Interface to Stick to the Core and Expose the Resulting Builder

Testing the string results of the renderer won't cut it. I'd have to duplicate template assertions all over again. With variations of the table body. It'd be a mess of essentially duplicate test cases across 3 test suites. Then there's another solution, equally unattractive: rewriting the objects to be reference type objects (`class`) and using mocks to verify the renderer delegates properly.

Thinking about stuff I read in the past couple of weeks about declarative interfaces and how value objects _are_ data things instead of behavior-rich classes, I came up with a different solution.

I made the `LatexExportBuilder` equatable; the actual `build()` call isn't very interesting from the renderer's point of view. It matters that calling `build()` produces the same result for similar configurations. That's what its tests ensured. When two builder configurations are equal, the resulting string is going to be equal, too.

This opened up a new way to test the renderer. Instead of writing assertions about the resulting string, I could write assertions about the resulting builder configuration.

That means I won't duplicate the actual string rendition tests. And I won't need mocks, either.

## Minimizing the Integration Layer (and not testing it at all)

The renderer becomes a "configurator". Client code obtains a builder and can even alter the default configuration if needed. Tests verify the correct setup. All is good and well with this functional core.

I stick with "renderer", though. A convenience `render()` function returns the result of `builder.build()`. That's a single line which I will not even need to test. The core functionality is well-tested, and this super thin shell around the core will not get better when I write integration tests for it. It's so tiny that its impact is negligible. And [integration tests are a scam anyway](https://vimeo.com/80533536).

So that's this declarative power everyone talks about. Thinking in terms of good old OO design, I was stuck with thinking in terms of writing assertions about the delegation itself: assertions for the method calls of collaborating objects. With a declarative, black-box, input--output-mindset, I could instead use value objects to encapsulate and represent the transformation steps -- and write assertions for them instead of the primitive string result.

