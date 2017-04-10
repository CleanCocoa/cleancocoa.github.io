---
title: Plug-in a UITextView Placeholder Label Extension
created_at: 2015-02-22 12:18:16 +0200
kind: worklog
tags: [ extension, swift, view-controller ]
image: 201502221217_screenshot.png
comments: on
code: swift
---

Kevin McNeish [created a Swift extension][tut] for `UITextView` to show a placeholder text when a text field is still empty. It takes all the burden from the view controller and is a real plug-in: it won't interfere with existing code and only requires you add the file to your project. That's it.

While the extension is useful in itself, I think the way Kevin implemented it is even more interesting: we can learn how to use helper objects which are invisible to the client code to plug-in new behavior into existing objects -- and even in a fairly complex manner, for that matter, since Kevin utilizes `NSNotificationCenter` to show and hide the label!

So this little extension is fairly involved behind the scenes. Still, it's pretty straightforward. Have a look at Kevin's sequence diagram to see the flow of information:

{% include figure.md src="/assets/blog/2015/201502221215_placeholder_label.jpg" alt="Sequence Diagram" caption="Sequence diagram, showing that setting the new placeholder attribute will (2) create a new label, (5/6) create and register a private notification handler, and (10) show/hide the label appropriately." %}

This extension does two things: it adds a placeholder label on top of the `UITextView` if necessary and makes it respond to user interaction with the label.

Usually, and by "usually" I mean in the Apple-recommended way shown in example code and the docs, you'd place this logic in your view controllers. They, in turn, get convoluted with handling their sub view's needs. Almighty view controllers make you app hard to extend and debug.

So Kevin's extension really does one thing for you: it makes your code more readable and maintainable by taking user interaction logic out of the view controllers.

This would be more difficult to do as a category in Objective-C because of the property observers Kevin used. But well, that's the power of Swift.

To create the placeholder, the extension adds an [`@IBInspectable`][inspectable] property, `placeholder: String?`. It's just a wrapper around the private optional `placeholderLabel: UILabel?` this extension also adds. When you set the placeholder value, the label is created. If you don't use it, nothing changes. So this extension really affects only those instances of `UITextView` where you set a placeholder value in Interface Builder or in code.

Next to the placeholder, this extension also adds a `NotificationProxy` which responds to user interaction to show and hide the placeholder via `UITextViewTextDidChangeNotification`.

`NotificationProxy` is just a super-thin container for an `NSObjectProtocol` object which itself is just a container for block-based notification handling. Since deallocating `NSObjectProtocol` instances won't cause them to de-register themselves from their associated notification center, you need the proxy primarily to call `removeObserver(_:)` on `deinit`, which is the new `dealloc` in Swift.

`NotificationProxy`, slightly rephrased in my own terms, looks like this:

```swift
class NotificationProxy: UIView {
    
    weak var notificationHandler: NSObjectProtocol!
    
    lazy var notificationCenter: NSNotificationCenter! = {
        return NSNotificationCenter.defaultCenter()
    }()
    
    func addObserverForName(name: String?, object: AnyObject?, queue: NSOperationQueue?, usingBlock: (NSNotification!) -> ()) {
        notificationHandler = notificationCenter.addObserverForName(name, object: object, queue: queue, usingBlock: usingBlock)
    }

    deinit {
        notificationCenter.removeObserver(notificationHandler)
    }
}
```

It's a descendant of `UIView` so the extension can keep a strong reference to this helper object by using `addSubview(_:)`. It's a little hacky, but it works.

Take a look at [Kevin's article][tut] to learn more and get the example code.

Solutions like this make your code more modular, put your view controllers on a diet, and may enhance the overall maintainability of your code. Aside from making `NotificationProxy` inherit from `UIView`, which is wrong to me semantically, I really love this simple solution.

[tut]: http://www.iphonelife.com/blog/31369/swift-programming-101-creating-self-registering-swift-ui-controls
[inspectable]: https://developer.apple.com/library/ios/recipes/xcode_help-IB_objects_media/chapters/CreatingaLiveViewofaCustomObject.html
