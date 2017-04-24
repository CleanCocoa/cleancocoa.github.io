---
title: Swift Protocols with Default Implementations as UI Mixins
created_at: 2017-04-24 15:56:50 +0200
tags: [ protocol, mixin ]
url: https://gist.github.com/odrobnik/ae16f4f071ead51d915712818a2279d8
comments: on
---

Oliver Drobnik of [Cocoanetics](https://www.cocoanetics.com) posted a little gist about a protocol that offers pull-to-refresh for your `UITableView`s: <https://gist.github.com/odrobnik/ae16f4f071ead51d915712818a2279d8>


```swift
@objc protocol Refreshable
{
    /// The refresh control
    var refreshControl: UIRefreshControl? { get set }
    
    /// The table view
    var tableView: UITableView! { get set }
    
    /// the function to call when the user pulls down to refresh
    @objc func handleRefresh(_ sender: Any);
}


extension Refreshable where Self: UIViewController
{
    /// Install the refresh control on the table view
    func installRefreshControl()
    {
        let refreshControl = UIRefreshControl()
        refreshControl.tintColor = .primaryColor
        refreshControl.addTarget(self, action: #selector(handleRefresh(_:)), for: .valueChanged)
        self.refreshControl = refreshControl
        
        if #available(iOS 10.0, *)
        {
            tableView.refreshControl = refreshControl
        }
        else
        {
            tableView.backgroundView = refreshControl
        }
    }
}
```

I use protocols for interface abstractions in my model and service layer a lot; but in the UI, I often resort to delegation to classes. I almost never use protocols in the UI that come with default implementations. Now that I think about it, I believe most custom protocols that I do implement in the UI layer are of `DisplaysBananas` kind -- implementations for the view protocol of a presenter. 

Oliver's gist made me think about other techniques to separate concerns in the UI. Not just delegation to sub-view controllers and encapsulation of view data in structs, but maybe a few more protocol abstractions here and there. When they make sense, that is.
