---
title: Model-View-View Model in Swift
created_at: 2014-12-19 17:32:15 +0100
kind: worklog
tags: [ design-pattern, mvvm ]
url: http://rasic.info/bindings-generics-swift-and-mvvm/
comments: on
---

[Srdan Rasic][sr] wrote a good post on creating wrappers around value objects (like Strings, say) and custom-glue them to an application's interface. It's a technique I have used in [my book on Mac architecture][book] at one point but which seems to be tremendously useful for virtually anything view-related once you get to a high enough level of abstraction.

It's weird that articles related to programming are so hard to quote. Instead of a quotation, let the resulting code illustrates what he has to say:

    class ArticleViewController {
      var bodyTextView: UITextView
      var titleLabel: UILabel
      var dateLabel: UILabel
      var thumbnailImageView: UIImageView
   
      var viewModel: ArticleViewViewModel {
        didSet {
          viewModel.title.bindAndFire {
            [unowned self] in
            self.titleLabel.text = $0
          }
       
          viewModel.body.bindAndFire {
            [unowned self] in
            self.bodyTextView.text = $0
          }
       
          viewModel.date.bindAndFire {
            [unowned self] in
            self.dateLabel.text = $0
          }
       
          viewModel.thumbnail.bindAndFire {
            [unowned self] in
            self.thumbnailImageView.image = $0
          }
        }
      }
    }

So now when `viewModel`'s properties change, the view controller will update the corresponding interface component. Sweet.

<%= advertise :macappdevbook %>

[sr]: http://rasic.info/bindings-generics-swift-and-mvvm/
[book]: /posts/2014/12/exploring-mac-app-development-strategies-released/
