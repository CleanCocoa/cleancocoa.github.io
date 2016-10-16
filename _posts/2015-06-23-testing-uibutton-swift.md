---
title: How to Write Unit Tests for Storyboard-based Buttons in Swift
created_at: 2015-06-23 09:10:03 +0200
kind: worklog
tags: [ testing, unit-test, view-controller, swift ]
image: 201506230913_test-storyboard.png
comments: on
---

I'm writing unit tests for my Storyboard-based view controllers. Button interaction can be tested in view automation tests, but that's slow, and it's complicated, and it's not even necessary for most cases.

To write unit tests instead of UIAutomation tests for buttons, you test multiple things.

1. Check that the button is initialized from the Storyboard.
2. Verify that the button is wired to an action of the view controller.
3. Test that the button's label is set up correctly.
4. Test that invoking the action method produces the desired result.

Here are the tests.

    #!swift
    func testFinishButton_IsInitialized() {
        
        XCTAssert(hasValue(viewController.finishButton))
    }
    
    func testFinishButton_IsLabelledCompleteAndLog() {
        
        XCTAssert(viewController.finishButton?.titleLabel?.text 
            == "Complete and Log")
    }
    
    func testFinishButton_IsWiredToAction() {
        
        let actions = buttonActions(viewController.finishButton)
        
        XCTAssert(hasValue(actions))
        if let actions = actions {
            XCTAssert(actions.count > 0)
            XCTAssert(contains(actions, "finishExercise:"))
        }
    }
    
    func buttonActions(button: UIButton?) -> [String]? {
        
        return button?.actionsForTarget(viewController, forControlEvent: .TouchUpInside) as? [String]
    }

Then you have to add a usual unit test for the action method, `finishExercise:` in this case. And then you're set: there's nothing left UIAutomation tests can do for you to verify the basic behavior of the button.

Check [Jon Reids screencasts](http://qualitycoding.org/uiviewcontroller-tdd/) to find out more about testing view controllers in general or have a look at [the article](http://www.objc.io/issues/1-view-controllers/testing-view-controllers/) by Daniel Eggert at objc.io.
