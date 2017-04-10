---
title: RocketData is a Uni-Directional UI Component Data Source
created_at: 2016-11-13 15:34:22 +0100
kind: worklog
tags: [ vipes, 1df, reswift, network ]
vgwort: http://vg01.met.vgwort.de/na/fae7ea7c6eb2419284795857c4dea4a9
comments: on
---


I watched ["Managing Consistency of Immutable Models" by Peter Livesey][pl] where Peter shows how [RocketData](https://github.com/linkedin/RocketData) works. Very worth the time!

I'd call RocketData a **uni-directional data source**. It's uni-directional because you set up the `DataProvider` as the source once, wire update notifications to view updates via the `delegate` property, and you're done with the setup. Update events include:

* Overwriting data, as you'd do to push the result of a successful network request to the `DataProvider`, 
* Request old data from cache, 
* or sync one `DataProvider` instance with another, maybe across different view controllers (think `UISplitViewController`!).

It's all the same to the `DataProvider.delegate`, which you'll most likely implement in the view controller. (Or in your Presenter object, if you have any. While we're at it, the `DataProvider` behaves like a VIPER Interactor which you wire to a Presenter. If the task is trivial, Interactor, Presenter, and View collapse into a single component that uses RocketData.) The delegate's `dataProviderHasUpdatedData(_:, context:)` callback will be triggered.

An example from Peter's slides:

    #!swift
    class MyViewController: UIViewController {
        let dataProvider = DataProvider<PersonModel>()

        func viewDidLoad() {
            super.viewDidLoad()
            dataProvider.fetchDataFromCache(cacheKey: self.id) { (_, _) in
                self.refreshView()
            }
            MyNetworkManager.fetchPerson(id: self.id) { (model, error) in
                if let model = model {
                    self.dataProvider.setData(model)
                    self.refreshView()
                }
            }
        }
    }

Here you see that the `DataProvider` queries the cache (which is supposed to quickly find results on a background thread from a local key--value-store) and updates the view when finished, and that a network request is fired to fetch new data and push changes to the view, too.
    
A specialized `CollectionDataProvider`, your go-to table view data source, will uses a different delegate that [passes change information along](https://github.com/linkedin/RocketData/blob/master/RocketData/CollectionChange.swift), essentially:

    #!swift
    enum CollectionChangeInformation {
        case update(index: Int)
        case delete(index: Int)
        case insert(index: Int)
    }

So it's uni-directional because `DataProvider` updates are pushed to the delegate (your view component). In order to update the user interface, you have to push model changes either to the `DataProvider` directly or to a synced `DataProvider` instance. [Models](https://github.com/linkedin/RocketData/blob/master/RocketData/Model.swift) are designed to be immutable, so you cannot change data under the feet of your UI.

RocketData is a per-view component data source. The underlying principles are very similar to what I got to know from [ReSwift](/posts/tags/reswift/). Bringing RocketData and ReSwift into context, you can say ReSwift is a `DataProvider` for all of your app. RocketData focuses on keeping single components consistent; ReSwift focuses on keeping all of your app's state consistent. 

I'd love to see their cache implementation, too. The combination of a key--value-cache and RocketData can be very powerful if your app relies on data from the network a lot.

[pl]: https://realm.io/news/slug-peter-livesey-managing-consistency-immutable-models/
