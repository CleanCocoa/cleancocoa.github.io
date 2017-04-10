---
title: Save Your Butt in Unit Tests With Shallow Reaching
created_at: 2015-03-25 17:46:00 +0100
kind: worklog
tags: [ testing, law-of-demeter ]
url: http://www.theswiftlearner.com/2015/03/05/the-law-of-demeter-and-the-9th-circle-of-hell/
comments: on
code: swift
---

I assume you _do_ write tests. To test system boundaries, you have to find out whether methods are called. Usually, people reach out for mocks to verify behavior.

If you use a lot of mocks, though, the tests can [reveal high coupling in your code](https://www.destroyallsoftware.com/blog/2014/test-isolation-is-about-avoiding-mocks): either you have to mock objects multiple levels deep, which is a sign of too much knowledge about how the collaborating objects work, or you have to mock lots of objects one level deep. The latter can be acceptable when you test a components which is some kind of coordinator. The former, though, is never to be accepted. Use the feedback of these hard-to-write tests to make your code simpler.

That's what the [Law of Demeter][lod] is for. While "law" is way too strict, its _principle_ is sound: strive to call methods on the objects you have direct access to, but avoid calling methods on objects which you obtain through another method call in the first place.

For example, the following violates the principle:
    
```swift
func violateThePrinciple(aCollaborator: Collaborator) {
    let aStranger = aCollaborator.obtainFriend() // this is okay so far
    aStranger.doSomething() // this is not, because it's 2nd level
}
```

You can get by through [providing facade methods](http://www.theswiftlearner.com/2015/03/05/the-law-of-demeter-and-the-9th-circle-of-hell/):

```swift
func respectThePrinciple(aCollaborator: Collaborator) {
    aCollaborator.doSomething()
}

extension Collaborator {
    func doSomething() {
        self.friend.doSomething()
    }
}
```

In your unit tests, you can add a `Collaborator` mock to verify the calls and then you're good to go. No multiple levels of mock objects and stubs. You may call this "shallow reaching" (opposed to "deep reaching"), although no one else calls it that.

That's the Law of Demeter in a nutshell: only talk to friends which you know through properties or parameters, but don't talk to strangers which you get to know through the return values of your friends' methods. You never know what to expect.

[lod]: http://en.wikipedia.org/wiki/Law_of_Demeter
