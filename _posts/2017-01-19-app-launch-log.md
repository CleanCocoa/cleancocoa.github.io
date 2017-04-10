---
title: A Look at the ReSwift Event Log During Launch of My Latest Project
created_at: 2017-01-19 09:04:38 +0100
kind: worklog
tags: [ reswift, middleware ]
comments: on
---

I use a logging middleware in all my apps that use ReSwift. I love to see the event log. It makes me think the app is healthy when it tells me what's going on.

All this happens when I start the knowledge management app I'm working on in about half a second (reformatted a bit to make reading easier):

    > ReSwiftInit()
    > Routing(route: .launch)
    > ChangingArchiveConfiguration(
        configuration: Optional(ArchiveConfiguration(folder=Folder(path="/Users/ctm/Pending/Zettelkasten-App/testdata"), 
        extensions=PlainTextExtensions())))
    > InitializingArchive.started
    TODO: show loading indicator
    > Routing(route: .mainWindow(MainRoute(
        state=State(
            searchResults: ArchiveContents([], []), 
            noteText: nil, 
            showSettings: false), 
        components=Components(
            search: Search(archive=Archive(<InMemoryZettelFinder>), lastState=nil), 
            display: DisplayNote(), 
            indexing: Indexing(), 
            writing: WriteFileChanges())))
    > SearchingNotes(query: Query("", context=[]))
    > SearchingNotesFinished(ArchiveContents([§id2, §id], []))
    > DisplayingNoteFinished.displayNone
    > InitializingArchive.successful
    TODO: close loading indicator
    error indexing: noNote(file:///Users/ctm/Pending/Zettelkasten-App/testdata/.DS_Store)
    > ReloadingResults(trigger: note changed in repository (§201201301230))
    > SearchingNotesFinished(ArchiveContents([§id2, §id, §201201301230], []))
    > ReloadingResults(trigger: note changed in repository (¿this is a proto zettel?))
    > SearchingNotesFinished(ArchiveContents([§id2, §201202011545, §201202011557, §201202092218, §201202011633, §201202011636, §201201311533, §201202011539, §201202011536, §201202011542, §201202011602, §201202092225, §201202101433, §201202101439, §201201301230, §id, §201202011550, §201202011607, §201202011623, §201201311302, §201201311311, §201612241053], [¿this is a proto zettel?]))

The bottom part tells me the initial note indexing sends `ReloadingResults` events correctly. It is throttled already to fire only every 0.2 seconds, but that still means it will fire at least twice: once in the beginning (that's the first note with the ID "201201301230") and later when it's done. Maybe I'll get rid of a live-updating list of files when a lot of work is performed.

TableFlip's log looks similar. Here's what happens when TableFlip launches with 2 windows open. You can see that in `/tmp/tableflip.log`, too, if you run the app:

    > ReSwiftInit()
    > ReSwiftInit()
    > ChangeLock(isLocked: false)
    > ChangeLicenseInformationAction(
        licenseInformation: TrialLicense.LicenseInformation.registered(
            TrialLicense.License(name: xxxx, licenseCode: xxxx)))
    > ChangeLock(isLocked: false)
    > ReSwiftInit()
    > ReSwiftInit()
    > ChangeLock(isLocked: false)
    > Undone with SelectTable(tableIndex: 0)
    > ChangeLicenseInformationAction(
        licenseInformation: TrialLicense.LicenseInformation.registered(
            TrialLicense.License(name: xxxx, licenseCode: xxxx)))
    > ChangeLock(isLocked: false)

That, in turn, tells me I can save a few cycles by optimizing the locking/unlocking of documents based on license information. `Undone with SelectTable(tableIndex: 0)` is an artifact of resetting the `NSDocument` to display the initial table data properly by making sure the first table in the document is selected. "Undone" here indicates it's not an action that is undoable. (See my [Undo middleware](/posts/2016/08/reswift-undo-middleware/).)

This is what happens when you move the cursor twice to the right in TableFlip:

    > selectCell(Selection(1, 1)) @ 0
    > selectCell(Selection(2, 1)) @ 0
    > selectCell(Selection(3, 1)) @ 0

Then type something and hit return, triggering an autosave in the process:

    > changeCell((#3, #1), 'hello!') @ 0
    > selectCell(Selection(3, 2)) @ 0
    Determining write operation for net.daringfireball.markdown
    Changed document type to canonical Optional("net.daringfireball.markdown")
    Requesting data of type net.daringfireball.markdown for document of Optional(TableFlip.DocumentType.markdown)

I hate dealing with file types, by the way.

Here's the logging middleware, in case you want to have as much fun as I:

    #!swift
    import Foundation
    import ReSwift
    
    let formatter: DateFormatter = {
        let formatter = DateFormatter()
        formatter.dateFormat = "HH':'mm':'ss'.'SSSS"
        return formatter
    }()

    func log(_ text: String) {
        let time = formatter.string(from: Date())
        print("\(time) \(text)")
    }

    let loggingMiddleware: Middleware = { dispatch, getState in
        return { next in
            return { action in
                log("> \(action)")
                return next(action)
            }
        }
    }
