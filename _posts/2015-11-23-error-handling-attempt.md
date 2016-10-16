---
title: How to Handle Errors When You're Not Interested in the Details
created_at: 2015-11-23 08:55:23 +0100
kind: worklog
tags: [ error-handling ]
comments: on
---

I read [a post by Erica Sadun][es] about making `try?` more useful. This new variant of the keyword transforms thrown errors into `nil`. That can be convenient, but it doesn't help handle the errors. Erica proposes a wrapper which logs errors. I like that idea and take it a step further.

Erica's functions are pretty awesome as is. Have a look at how they're used:

    #!swift
    // Becomes nil on error
    let contents = attempt { try NSFileManager.defaultManager()
        .contentsOfDirectoryAtPath(fakePath) }
    let maybeWorked = attempt(potentiallyThrowingMethod())
    
    // Returns false on error
    let success = attemptFailable { try "Test".writeToFile(fakePath, 
        atomically: true, encoding: NSUTF8StringEncoding) }

Erica prints errors when they happen but doesn't do much else with them in the sample codes. I bet she does in the resulting apps, though. The function definitions look like this at the moment:

    #!swift
    public func attempt<T>(source source: String = __FUNCTION__, file: String = __FILE__, line: Int = __LINE__, closure: () throws -> T) -> Optional<T>{
        do {
            return try closure()
        } catch {
            let fileName = (file as NSString).lastPathComponent
            let report = "Error \(fileName):\(source):\(line):\n    \(error)"
            print(report)
            return nil
        }
    }

    public func attemptFailable(source source: String = __FUNCTION__, file: String = __FILE__, line: Int = __LINE__, closure: () throws -> Void) -> Bool {
        do {
            try closure()
            return true
        } catch {
            let fileName = (file as NSString).lastPathComponent
            let report = "Error \(fileName):\(source):\(line):\n    \(error)"
            print(report)
            return false
        }
    }
    
Now what if you dispatched application errors to the user?

Especially most Core Data errors are in fact programmer errors. When they occur, you're in charge to stop them from happening. File read errors or network connection failures are different. Failure on these fronts should be handled well by the application, but there's no need to report every lost network connection to the developer.

An enhancement of Erica's methods for production apps would be:

* replace `print` with some kind of logging
* send a message that an application-level error occured; use these for reporting

I [wrote](/posts/2015/10/error-handler-appkit/) about my global `applicationError` helper function and the `ErrorHandling` module ([on GitHub](https://github.com/DivineDominion/ErrorHandling)) already. The `applicationError` function is great to present misbehavior of the app to the user so she can report problems. 

My Swift module will take care of the last point. Leaves logging.

Now I don't want to implement my own logging framework. But I think we can register custom loggers if they adhere to a common signature:

    #!swift
    // from https://github.com/DivineDominion/ErrorHandling#convenience-methods
    class ErrorHandler {
        
        typealias Logger = (format: String, args: CVarArgType...) -> Void

        static var log: Logger = NSLog
        
        let log: Logger
        
        init() {
            self.log = ErrorHandling.log
        }
        
        // ...
    }
    

The `Logger` signature is compatible to CocoaLumberjack and similar, too. It's easy to set up the `ErrorHandling` module for logging if I add that line.

    #!swift
    // Convenience function
    public func attempt<T>(source source: String = __FUNCTION__, file: String = __FILE__, line: Int = __LINE__, closure: () throws -> T) -> Optional<T>{
        
        let errorHandler = ErrorHandler()
        errorHandler.attempt(source: source, file: file, line: line, closure: closure)
    }
    
    extension ErrorHandler {
        func attempt<T>(source source: String = __FUNCTION__, file: String = __FILE__, line: Int = __LINE__, closure: () throws -> T) -> Optional<T>{
            do {
                return try closure()
            } catch {
                let error = ErrorHandler.errorWithMessage(message: nil, function: source, file: file, line: line)
                handle(error)
                
                return nil
            }
        }
    }
    
Recall that the `ErrorHandler` from my module's repository does not much more than logging, showing an alert, and creating `NSError`s:

    #!swift
    extension ErrorHandler {
        public func handle(error: NSError?) {

            if let error = error {
                logError(error)
                reportError(error)
            }
        }
        
        private func logError(error: NSError) {

            log("Error: \(error)")
        }

        private func reportError(error: NSError) {

            ErrorAlert(error: error).displayModal()
        }
        
        public static func errorWithMessage(message: String? = nil, function: String = __FUNCTION__, file: String = __FILE__, line: Int = __LINE__) -> NSError {

            var userInfo: [String: AnyObject] = [
                functionKey: function,
                fileKey: file,
                lineKey: line,
            ]

            if let message = message {
                userInfo[NSLocalizedDescriptionKey] = message
            }

            return NSError(domain: exceptionDomain, code: 0, userInfo: userInfo)
        }
    }

The `ErrorAlert` takes care of displaying problems to the user. That's where formatting the report should take place; the part in question from Erica's original source:

    #!swift
    let fileName = (file as NSString).lastPathComponent
    let report = "Error \(fileName):\(source):\(line):\n    \(error)"


[es]: http://ericasadun.com/2015/11/05/implementing-printing-versions-of-try-and-try-on-steroids-in-swiftlang/
