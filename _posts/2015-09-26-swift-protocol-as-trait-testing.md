---
title: "Swift Protocol Extensions as Mixins – And How Do You Test That?"
created_at: 2015-09-26 12:14:17 +0200
kind: worklog
tags: [ swift, protocol, testing, mock ]
vgwort: http://vg08.met.vgwort.de/na/a94f926d9fb44e7a918b891c57fbb361
comments: on
---

I found a very useful distinction [by Matthijs Hollemans][matt] to grasp what protocol extensions in Swift 2 can be: instead of interfaces, they are traits or even mixins. At the same time, I want to raise awareness for misuse: just because behavior is mixed-in doesn't mean it's additional behavior. The code may live in another file, but its functionality can still clutter your objects.

Let's get the terms straight first:

> First off we have the **interface**. This is a protocol that just has method signatures but no actual code. This is what Objective-C and Swift 1.2 have.
> 
> A **trait** also has the actual method bodies. In other words, a trait adds code to an interface. This is the new thing that Swift 2.0 lets us do today with protocol extensions.
> 
> A **mixin** is like a trait but it also has state. We can’t really do this yet with Swift 2.0 as you’re not allowed to add stored properties to protocols.

With protocols including protocol extensions, you can provide default behavior. Instead of creating a helper object to delegate to, you can "mix in" (not in the technical term above) the behavior into objects.

Instead of this:

    #!swift
    class LoginViewController: UIViewController {
        let usernameValidator = UsernameValidator() 
        let passwordValidator = PasswordValidator()
        // ...
    }

You get this:

    #!swift
    class LoginViewController: UIViewController, ValidatesUsername, ValidatesPassword {
        // ...
    }

These mixes-in the methods `isUsernameValid(_:) -> Bool` and `isPasswordValid(_:) -> Bool` to use in the view controller.

Matthijs likes that:

> This is a nice trick to keep your view controllers clean without having to use extra helper objects. I think this is pretty cool. And you can also easily reuse these mixins in other view controllers and in other projects.

_But beware of mixing responsibilities!_

In a better-than-average-but-not-ideal world already, you'd write tests for the `LoginViewController` to verify its behavior.

How do you do it if the view controller does everything? You provide a good user name, a bad user name, a good password, a bad password, and then combinations of these and see if the login action is triggered. If you display "You can't leave this blank" messages independent from validation logic, you have to test that as well. And everything else the view controller does. Which is probably a lot more than you'd like.

If you delegate to a `LoginViewControllerValidator` which is tailor-made for the `LoginViewController`, say, then you only have to verify that whatever's in the text fields is forwarded to `LoginViewControllerValidator`. That's just two assertions, one for each text field.

You can remedy part of the pain [by mocking the tested object itself](/posts/2015/09/test-mocked-classes/):

    #!swift
    class MockLoginViewController: LoginViewController {
    
        var testUsernameValidity = false
        var didValidateUsernameWith: String? 
        override func isUsernameValid(username: String) -> Bool {
          
            didValidateUsernameWith = username
            
            return testUsernameValidity
        }
        
        // ... same for password ...
    }
    
    // ...
    
    let controller = MockLoginViewController()
    
    func testLogin_ValidatesUserName() {
        
        let username = "a name"
        controller.usernameTextfield.text = username
        
        controller.login()
        
        XCTAssert(controller.didValidateUsernameWith == username)
    }
    
    // ...
    
    func testLogin_WithValidUserNameAndPassword_UnlocksTheApp() {
        
        // No need to set actual text field contents since the 
        // mocked object doesn't really use them anyway. Just set
        // the outcome like you would if this were a mocked dependency.
        controller.testUsernameValidity = true
        controller.testPasswordValidity = true
        
        controller.login()
        
        XCTAssert(controllerDelegateDouble.didUnlockApp)
    }

Imagine how many fields a `SignUpFormController` may contain. Mixing all that in is going to make your view controller's test suite files very long. Or you don't test at all, because it's getting on your nerves.

No matter how bad a reputation delegation to helper objects has among Objective-C veterans: with Swift, the overhead to create a new class is next to zero while the flow of information is getting so much easier to test and reason about.

So don't write the next `class Banana: Fruit, Peelable, Edible, TastesGoodInSmoothies, DecaysInAWeek, StartsWithYellowColor, DecaysToBrownColor, CanBeThrown { ... }`. Consider the payoff between coding quickly and creating reliable code.

Above all, _write tests._

[matt]: http://matthijshollemans.com/2015/07/22/mixins-and-traits-in-swift-2/
