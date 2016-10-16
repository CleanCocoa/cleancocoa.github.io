---
title: Currying and Treating Instance Methods as Curried Functions
created_at: 2016-03-31 14:25:10 +0200
kind: worklog
tags: [ currying, swift, closure ]
vgwort: http://vg01.met.vgwort.de/na/fb42e4e0c04148e4b6739412f0909a64
comments: on
---


Currying is a very useful and interesting thing. It means that you can write functions in parts. If you don't provide all parts at once, you won't get the "real" result but a closure that accepts the missing parameters.

Like this, with spacing added to reveal how the nested closure maps to the chain of return types:

    #!swift
    func add(a: Int) -> (Int)    -> Int {
        return {        (b: Int) -> Int in
            return a + b
        }
    }

    print(add(31)(13)) // => 44

So you can break the parameter list up and call the result of the first function call again. This is useful for storing partial applications:

    #!swift
    let add5 = add(5)
    print(add5(10))    // => 15
    print(add5(20))    // => 25

This is turn can be useful to provide context for setting up a block to be called later. You provide the parameters you know in advance and leave some out to be called later.

    #!swift
    class Server {
        func fetchData(completion: (data: NSData?) -> Void) {
            //...
            completion(result)
        }
    }

This method expects a block or function of type `NSData -> Void`. Here's a curried function that does not fit unless you provide the first set of parameters:

    #!swift
    func showDataInViewController(viewController: ServerDataViewController) -> (NSData?) -> Void {
        return { data in 
            // This closure captures `viewController`, too.
            guard let data = data else { return }
            viewController.showData(data)
        }
    }

Now I here's a contrived example to show how you can vary one part of that function call while leaving the other unspecified
    
    #!swift
    if someInitialCondition {    
        server.fetchData(showDataInViewController(initialViewController))
    } else {
        server.fetchData(showDataInViewController(historyViewController))
    }

You see:

* `showDataInViewController(foo)` is of type `(NSData) -> Void`, while
* `showDataInViewController` is of type `(ServerDataViewController) -> (NSData?) -> Void`.

If all this is super new to you: you can reference any function or method instead of calling it, like so:

    #!swift
    // Reference print(...)
    let logger = print
    
    // Use the reference later
    logger("hi")   // => hi

Now to the fun part which I haven't used until today: you can do the same with methods when you don't know the object, yet.

Sticking to the example from above:

    #!swift
    class Server {
        func fetchData(completion: (data: NSData?) -> Void) { /* ... */ }
    }
    
    let fetchDataFromServer = Server.fetchData

When you have an object, the following calls will produce the same result:

    #!swift
    let aServer = Server()
    fetchDataFromServer(aServer) { data in
        // ...
    }
    aServer.fetchData() { data in
        // ...
    }

You'll see:

* `fetchDataFromServer` is of type `(Server) -> ((NSData?) -> Void) -> Void`, while
* `server.fetchData` is of type `((NSData?) -> Void) -> Void`.

This looks just like what we tried with currying from above. Even though this might not even be the same mechanism, from a pragmatic point of view calling `ClassName.methodName` does the same to the method like creating a curried static variant of it.

## Real-world example

I used it to produce function pointers instead of delegating directly. Say I have over-specified my methods somewhere, like so:

    #!swift
    extension Coordinates {
  
        func moveUp() -> Coordinates {
            // y - 1
        }
  
        func moveDown() -> Coordinates {
            // y + 1
        }
  
        func moveLeft() -> Coordinates {
            // y - 1
        }
  
        func moveRight() -> Coordinates {
            // y + 1
        }
    }

These explicit methods are a pain: [they over-specify the changes](http://www.natpryce.com/articles/000816.html) that are possible by defining methods for each of the directions. I'd prefer to type the directions instead to make the interface cleaner:
    
    #!swift
    enum Direction {
        case Up
        case Down
        case Left
        case Right
    }

To leverage the existing code before refactoring, I found a switch helpful that doesn't delegate from `Direction` back to `Coordinates`, at least not directly:

    #!swift
    extension Direction {
        
        var offsetCoordinates: (Coordinates) -> () -> Coordinates {
            switch self {
            case Up: return Coordinates.declinedRow
            case Down: return Coordinates.advancedRow
            case Left: return Coordinates.declinedColumn
            case Right: return Coordinates.advancedRow
            }
        }
    }
    
    extension Coordinates {
  
        func offsetIn(direction: Direction) -> Coordinates {
            return direction.offsetCoordinates(self)()
        }
    }
    
`offsetCoordinates` produces references to the overly explicit methods of `Coordinates`. Instead of requiring a reference to a coordinate object, the property doesn't need anything at all. The resulting closure will require a coordinate to pull the method down to the instance level again:

    #!swift
    direction.offsetCoordinates(self)

The result itself is a reference to one of the "move" methods and needs to be executed as well:

    #!swift
    direction.offsetCoordinates(self)()
    //                               ^^

Now I don't know when you're going to need stuff like this, but if you do, don't be afraid to try it out. 

In this particular case the intermediate solution was "clever" but inferior to defining a new `move(direction: Direction)` method with a new kind of `Direction` that can apply itself:

    #!swift
    struct Direction {
        static let Up    = Direction( 0, -1)
        static let Down  = Direction( 0,  1)
        static let Left  = Direction(-1,  0)
        static let Right = Direction( 1,  0)
        
        let offsetX: Int
        let offsetY: Int
        
        init(_ offsetX: Int, _ offsetY: Int) {
            
            self.offsetX = offsetX
            self.offsetY = offsetY
        }
    }
