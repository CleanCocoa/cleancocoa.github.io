---
title: Yelp's Modular UITableViews
created_at: 2015-12-10 19:17:27 +0100
kind: worklog
tags: [ uitableview ]
comments: on
---

Yelp reuses a setup of [modular table views][yelp] across their huge app. A simple timeline entry can be composed of 5 cells, each with its own model representation. These cells are used to create various component setups.

What I find most interesting is the combination of registering cells for reuse identifiers in one place and using `reuseIdentifierForCellAtIndexPath` in another.

The data source has to provide a `reuseIdentifierForCellAtIndexPath` method. This can be as simple as `NSStringFromClass([YLExampleTextCell class])`. I like that very much! The mapping from model data feels a bit weird at first glance, but maybe that's not the worst tradeoff ever.

    #!objc
    - (NSString *)tableView:(UITableView *)tableView reuseIdentifierForCellAtIndexPath:(NSIndexPath *)indexPath {
        id model = [self tableView:tableView modelForCellAtIndexPath:indexPath];
        if ([model isKindOfClass:[NSString class]]) {
            return NSStringFromClass([YLExampleTextCell class]);
        } else if ([model isKindOfClass:[YLExampleImagesCellModel class]]) {
            return NSStringFromClass([YLExampleImagesCell class]);
        }
        NSAssert(NO, @"No reuse identifier for model %@", model);
        return nil;
    }
    

Registering cells in the client code sounds very simple, too:

    #!objc
    @interface YLExampleViewController : UIViewController
    @end
    
    @implementation YLExampleViewController
    - (void)loadView {
      [super loadView];
      YLExampleTableViewDataSource *dataSource = [[YLExampleTableViewDataSource alloc] init];
    
      YLTableView *tableView = [[YLTableView alloc] initWithFrame:CGRectZero style:UITableViewStyleGrouped];
      // YLTableViewDataSource has to be the delegate and the dataSource of the table view.
      tableView.dataSource = dataSource;
      tableView.delegate = dataSource;
    
      // Registering cells and reuse identifiers
      [tableView registerClass:[YLExampleTextCell class] forCellReuseIdentifier:NSStringFromClass([YLExampleTextCell class])];
    
      self.view = tableView;
    }
    @end

This creates an implicit coupling: the cells have to be registered before they are used in code, and this view controller is the place where that happens. You can write unit tests to verify that a set of cells is registered [through mocking](/posts/2015/09/test-mocked-classes/), although that's not very convenient to do since `loadView` creates the table view that'll receive the `registerClass:forReuseIdentifier:` messages.

The only real drawback for me so far: it's written in Objective-C and works so great because a view model object can be `AnyObject`. I'm not a fan of that because you have to conditionally cast to the expected type. But it does the job and provide sufficient flexibility.

After all, maybe it's possible to replace this approach with generics to gain strict typing advantages. (Yelp's sample code runs `NSAssert`ions to check the type of the view model during runtime.)

[yelp]: https://github.com/Yelp/YLTableView


