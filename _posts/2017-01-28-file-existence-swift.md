---
title: Better Swift FileManager File Existence Checks
created_at: 2017-01-28 08:49:23 +0100
kind: worklog
tags: [ swift, file ]
vgwort: http://vg01.met.vgwort.de/na/be7dda24c23a4621a7ea49e905572120
comments: on
---

I found the Foundation way to check for file existence very roundabout. It returns 1 boolean to indicate existence and you can have another boolean indicate if the item is a directory -- passed by reference, like you used to in Objective-C land.

That's a silent cry for an enum. So I created a simple wrapper that tells me what to expect at a given URL:

```swift
public enum FileExistence: Equatable {
    case none
    case file
    case directory
}

public func ==(lhs: FileExistence, rhs: FileExistence) -> Bool {

    switch (lhs, rhs) {
    case (.none, .none),
         (.file, .file),
         (.directory, .directory):
        return true

    default: return false
    }
}

extension FileManager {
    public func existence(atUrl url: URL) -> FileExistence {

        var isDirectory: ObjCBool = false
        let exists = self.fileExists(atPath: url.path, isDirectory: &isDirectory)

        switch (exists, isDirectory.boolValue) {
        case (false, _): return .none
        case (true, false): return .file
        case (true, true): return .directory
        }
    }
}
```

And since you're probably interested in well-tested components, here're the unit tests:

```swift
class FileManager_ExistenceTests: XCTestCase {

    func testExistence_NonExistingFile() {

        let url = generatedTempFileURL()
        XCTAssertEqual(FileManager.default.existence(atUrl: url), FileExistence.none)
    }

    func testExistence_ExistingFile() {

        let url = generatedTempFileURL()

        guard let _ = try? "some content".write(to: url, atomically: false, encoding: .utf8)
            else { XCTFail("writing failed"); return }
        XCTAssertEqual(FileManager.default.existence(atUrl: url), FileExistence.file)
    }

    func testExistence_ExistingDirectory() {

        let url = createTempDirectory()
        XCTAssertEqual(FileManager.default.existence(atUrl: url), FileExistence.directory)
    }
}
```

... and here are the helper functions to create temporary files:

```swift
func createTempDirectory() -> URL {

    let fileName = "filemanagertest-temp-dir.\(NSUUID().uuidString)"
    let fileUrl = URL(fileURLWithPath: NSTemporaryDirectory()).appendingPathComponent(fileName)

    try! FileManager.default.createDirectory(at: fileUrl, withIntermediateDirectories: false, attributes: nil)

    return fileUrl
}

func generatedTempFileURL() -> URL {

    let fileName = "filemanagertest-temp.\(NSUUID().uuidString)"
    let fileURL = URL(fileURLWithPath: NSTemporaryDirectory()).appendingPathComponent(fileName)

    return fileURL
}
```

If you happen to have a better name than `existence(atUrl:)` come to mind, I'd be happy to change it!
