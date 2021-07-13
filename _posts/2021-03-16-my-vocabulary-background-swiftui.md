---
layout: post
title: Background and concurrent tasks in SwiftUI
subtitle: How to handle concurrency?
comments: true
readtime: true
hidden: true
---

For other kinds of background tasks in SwiftUI, for which the view must be updated after the task finishes. A `@State` or `@Published` property can be modified when the task is completed, but it is very important to do this in the main thread because everything that causes a UI update must be run in the main thread.

```swift
extension SpreadsheetView {
    class ViewModel: NSObject, ObservableObject {

        enum Status {
            case loading
            case importable
            case imported
        }

        @Published var status: Status = .loading

        func checkIfImported(translations: [Translation]) {
            DispatchQueue.global(qos: .background).async { [weak self] in
                for translation in self?.spreadsheet.translations ?? [] {
                    if !translations.contains(where: { $0.id == translation.id }) {
                        DispatchQueue.main.async { [weak self] in
                            self?.status = .importable // Will make the UI react, do it in main thread.
                        }
                        return
                    }
                }
                DispatchQueue.main.async { [weak self] in
                    self?.status = .imported // Will make the UI react, do it in main thread.
                }
            }
        }
    }
}
```

### Parsing spreadsheets concurrently

Parsing spreadsheets is a very demanding task that can take several seconds to complete, it is a good idea to parse all the spreadsheets concurrently to save some time.

```swift
func fetchAllSpreadsheets(completion: @escaping () -> ()) {
    // ...
    for file in (result as? GTLRDrive_FileList)?.files ?? [] {
        if let id = file.identifier, let name = file.name {
            self?.dispatchGroup.enter()
            print("Start fetching & parsing spreadsheet \(id)")
            self?.fetchSpreadsheet(id: id, name: name)
        }
    }
    self?.dispatchGroup.notify(queue: .main) { [weak self] in
        self?.spreadsheetsResult = .success(self?.spreadsheets ?? [])
        completion()
    }
}

func fetchSpreadsheet(id: String, name: String) {
    guard let url = spreadsheetUrl(id: id) else {
        dispatchGroup.leave()
        return
    }
    // Fetches and parses the content of each spreadsheet, given its ID.
    googleDriveService.fetcherService.fetcher(with: url).beginFetch { [weak self] data, error in
        guard error == nil, let data = data else {
            self?.dispatchGroup.leave()
            return
        }
        DispatchQueue.global(qos: .background).async { [weak self] in
            self?.generateSpreadsheet(from: data, id: id, name: name)
            self?.dispatchGroup.leave()
            print("Finish fetching & parsing spreadsheet \(id)")
        }
    }
}
```

#### DispatchGroup

Using `DispatchGroup` is a great choice because it is very easy to use. By calling `dispatchGroup.enter()` before every demanding task starts and `dispatchGroup.leave()` when it ends, when all the `dispatchGroup.leave()`'s have been called, the `self?.dispatchGroup.notify(queue: .main) { /* ... */ }` is executed.

Remember, if UI is going to be updated by the `notify` closure, it must run in the main thread.

In the code above, some `print` was added to see that the concurrent tasks may end in a different order to one in which they started.

```
Start fetching & parsing spreadsheet 1n4nY49aEwCGocULrdMWrhFr1JjKX5hxFjEssHKoEgIQ
Start fetching & parsing spreadsheet 1c6PcEbF65JgM_SU3rGxLe2Y1um_e7c5hNVIMZJ624S4
Start fetching & parsing spreadsheet 1zUDkpLmfgDUH4OfEferFQWT0R97HsX85ZKN8T0KibJg
Start fetching & parsing spreadsheet 11CWqCmMTK_Gy5ssT5ibZjkAirhkNeUZAH5sDshBqfFc

Finish fetching & parsing spreadsheet 1zUDkpLmfgDUH4OfEferFQWT0R97HsX85ZKN8T0KibJg
Finish fetching & parsing spreadsheet 11CWqCmMTK_Gy5ssT5ibZjkAirhkNeUZAH5sDshBqfFc
Finish fetching & parsing spreadsheet 1n4nY49aEwCGocULrdMWrhFr1JjKX5hxFjEssHKoEgIQ
Finish fetching & parsing spreadsheet 1c6PcEbF65JgM_SU3rGxLe2Y1um_e7c5hNVIMZJ624S4
```

<img src="../assets/img/my-vocabulary/concurrency/status_spreadsheets.jpg" width="280" class="center">


---

