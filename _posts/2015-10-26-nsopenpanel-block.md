---
title: NSOpenPanel's and NSSavePanel's Block-Based API is Superior
created_at: 2015-10-26 14:14:19 +0100
kind: worklog
tags: [ usability, east-oriented ]
comments: on
---

I got angry at `NSOpenPanel` the other day for visually blocking `NSAlert`s, and for staying in the way when the debugger hits a breakpoint. Silly me, I used ye olde `runModal`.

When you use the block-based API of `NSSavePanel` or its decendant `NSOpenPanel`, all of a sudden they will display on top, but only per application. Also, alerts seem to have a higher display level in these cases.

Of course now the client code has to be designed differently, too. Instead of waiting for a return calue, you pass a completion handler.

I don't think that's too bad, though:

    #!swift
    func addMonitorUsing(createMonitor: CreationStrategy) {

        pickedURLs() { URLs in

            let newMonitors = URLs.map({ NewMonitor(URL: $0, fileID: FileID()) })
    
            newMonitors.forEach(createMonitor)
        }
    }

    private func pickedURLs(completion: ([LocalURL]) -> Void) {

        return filePicker.selectedFilesWithExtensions(.Some(supportedFileExtensions), 
            completion: completion)
    }

Why, of course I wrapped the `NSOpenPanel` call in its own type:

    #!swift
    enum AllowedFileExtensions {
        
        case Any
        case Some([String])
    
        func preparePanel(panel: NSSavePanel) {
        
            panel.allowedFileTypes = self.allowedFileTypes
        }
    
        private var allowedFileTypes: [String]? {
            switch self {
            case .Any: return .None
            case let .Some(types): return types
            }
        }
    }

    class SelectFiles {
    
        init() { }
    
        func selectedFilesWithExtensions(fileExtensions: AllowedFileExtensions,
            completion: ([LocalURL]) -> Void) {
        
            let panel = NSOpenPanel()
            panel.canChooseDirectories = false
            panel.canChooseFiles = true
            panel.allowsMultipleSelection = true
        
            fileExtensions.preparePanel(panel)
        
            panel.beginWithCompletionHandler { response in
            
                guard response == NSFileHandlingPanelOKButton else {
                    return
                }
            
                let selectedFileURLs = panel.URLs.map() { LocalURL(URL: $0) }
                completion(selectedFileURLs)
            }
        }
    }

As a side-note, see how the `AllowedFileExtensions` enum helped me set up the panel while making the intent clear?

Initially, I commented the `selectedFilesWithExtensions` method thus:

    /// Show open file panel and return results as `LocalURLs`.
    ///
    /// - parameter fileExtensions: `nil` indicates "all file types"
    /// - returns: Collection of selected files; nil if cancelled.


But now I don't need to, because the types are self-documenting: the `AllowedFileExtensions` enum provides easy to understand options, and since the completion block is not called when nothing is selected, I don't even have to document a `nil` case for that, either.

When you allow `nil`, you essentially allow either all of the return type's values or something else entirely. I always try to get rid of it and make APIs simpler.

Now the app runs great, the user experience is better than before, and the panels get out of my way while debugging. I'm pretty happy with that now.
