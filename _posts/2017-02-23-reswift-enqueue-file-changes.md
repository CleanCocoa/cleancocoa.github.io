---
title: Handle Pending File Changes with ReSwift as Your Backbone
created_at: 2017-02-23 17:20:55 +0100
kind: worklog
tags: [ reswift ]
image: 20170117080653_filesaving.png
vgwort: http://vg01.met.vgwort.de/na/071baaf20557450a851fa3c74a60a7d4   
comments: on
---

Automatic saving of changes in some user interface component to a file should be handled differently when you employ ReSwift. 

{% include figure.md src="/assets/blog/2017/20170117080653_filesaving.png" alt="sketch of the information flow" caption="Your UI tells the app it finished editing. Then a reducer enqueues a pending file change." %}

In short, you have to extract the state information from the "save file" action and store it in the overall app's state somehow. I append these to a collection of `PendingFileChanges`, a `ReSwift.StateType` that is part of my overall app state.

(This is a follow-up to my [previous musings](/posts/2017/01/reswift-file-saving/) on the topic.)

"Save file" has the state info of "which file" and "what contents". In order to enqueue multiple changes to the same file, add a UUID to each file change to identify the event:

```swift
struct FileContentChange {
    let url: URL
    let content: String
    let uuid: UUID
}
```

Instances of this type are enqueued with the `PendingFileChanges` substate. I used real [FIFO queues](https://github.com/raywenderlich/swift-algorithm-club/tree/master/Queue) at first and solely relied on the order, but later switched to simple arrays. I'll tell you why in a minute.


```swift
struct AppState: StateType {
    var pendingFileChanges: PendingFileChangesState = .empty
}

struct PendingFileChangesState: StateType {

    static var empty: PendingFileChangesState {
        return PendingFileChangesState(fileContentChanges: [])
    }
    
    public fileprivate(set) var fileContentChanges: [FileContentChange]

    public var nextFileContentChange: FileContentChange? {
        return fileContentChanges.first
    }

    public mutating func insert(fileContentChange: FileContentChange) {
        fileContentChanges.append(fileContentChange)
    }

    public mutating func remove(fileContentChange: FileContentChange) {
        guard let index = fileContentChanges.index(of: fileContentChange) 
            else { return }
        
        fileContentChanges.remove(at: index)
    }
}
```

Some long-running service object will listen to changes to this substate and perform the necessary writes periodically. 

To simplify the setup, make the resulting `ChangeFileContents` service object a `ReSwift.StoreSubscriber`; now it'll receive updates to the app state from your store. Cache the last received (and thus pending) change in the service so you can make the operation asynchronous and allow more incoming state updates without issuing to overwrite the file time and again with the same stuff.

When it finishes a write operation, it in turn dispatches an action so the fitting item is removed from the queue.

Here's a sample service wireframe:
    
```swift
typealias DefaultStore = Store<AppState>

class WriteFileChanges: StoreSubscriber {
    let store: DefaultStore

    public init(store: DefaultStore) {
        self.store = store
    }

    fileprivate(set) var lastChange: FileContentChange?

    public func newState(state: PendingFileChangesState) {
    
        // If the state update shows the queue is empty, 
        // reset the cache.
        guard let change = state.nextFileContentChange else {
            lastChange = nil
            return
        }
        
        // Skip identical, unprocessed updates.
        guard change.uuid != lastChange.uuid else { return }
        
        // Set the cached value to prevent duplicate changes.
        lastChange = change

        delegatePerformingTheFileWriteOperation() { success in
            if !success { 
                // TODO: handle error :)
            }
            
            store.dispatch(CompletingFileContentChange(change))
        }
    }
    
    fileprivate func delegatePerformingTheFileWriteOperation(
        completion: (Bool) -> Void) {
        
        // Use FileManager or String.write(toURL:) etc.
        
        completion(true)
    }
}
```

So you end up with `WriteFileChanges` as a service that processes a part of the `PendingFileChangesState` queue. To let the app know when it has finished, i.e. let a reducer remove the entry from the pending changes queue, it dispatches a `CompletingFileContentChange` action. I leave this simple wrapper as an exercise to the reader.

Now that you know all of the code, here's why I ditched queues at first. With queues, I relied on the order of elements and used equality checks in the `WriteFileChanges` service to prevent doing the same operation twice. Before the UUID, enqueuing 2 equal `FileContentChange` objects resulted in the first one being processed, then popped from the queue after completion -- but then the second, identical object would never be processed. Now that I think of it, since adding the UUID, the issues I had with queues is solved. So it'd actually be a better idea to use a queue instead of an array to be clear about the order. _Plus_ make it a `UniqueElementQueue` or similar and add equality checks before adding elements so you don't end up enqueuing the same thing twice while lifting this detail into the type itself. 

A queue-based approach is a good idea because most apps will rely on the _order_ of file change events. If you have write and delete operations, you will want to put both into the same queue, too, by the way, and introduce a common type for `FileContentChange` and `FileRemoval`, say.

Having written that down, I'm going to change my implementation from array to queue with the UUID and see where this leads first thing next week.
