---
layout: post
title: Core Data in background
subtitle: Storing thousands of objects requires time...
comments: true
readtime: true
hidden: true
---

Saving just one or two items in Core Data is trivial, but when we need to store thousands of elements, the app can get stuck while all these items are added.

### Why must run in background?

```swift
for translation in spreadsheet.translations {
    guard self?.hasBeenImported(translation, in: translations) == true else { continue }
    let newTranslation = Translation(context: context)
    newTranslation.from = translation.from
    newTranslation.to = translation.to
    newTranslation.input = translation.input
    newTranslation.output = translation.output
}
```

This code works great for adding a few translations, but...

1. What would happen in the example above if the spreadsheet has 10.000 translations?
2. The `hasBeenImported` guard, verifies that the translation that is going to be added hasn't been added already. What happens when 20.000 translations have already been added?

### How to add thousands of items in background?

Running that code directly in the background using `DispatchQueue.global(qos: .background).async { /* ...*/ }` doesn't work for Core Data, to store the translations `performBackgroundTask` must be called on the `NSPersistentCloudKitContainer` to generate a new `NSManagedObjectContext` that will be able to perform background tasks.

```swift
func importTranslations(from spreadsheet: Spreadsheet, alreadyImported translations: [Translation]) {
    dataController.container.viewContext.automaticallyMergesChangesFromParent = true
    dataController.container.performBackgroundTask { [weak self] context in
    for translation in spreadsheet.translations {
        guard self?.hasBeenImported(translation, in: translations) == true else { continue }
        let newTranslation = Translation(context: context)
        newTranslation.from = translation.from
        newTranslation.to = translation.to
        newTranslation.input = translation.input
        newTranslation.output = translation.output
    }
    try? context.save()
}
```

There are 2 important lines in the code above:

1. Setting `dataController.container.viewContext.automaticallyMergesChangesFromParent = true`, the main context of the container (`viewContext`) will receive the changes immediately, even if the items are being written in other context.

2. The changes must be saved, but in the new context, with `try? context.save()`.

In other parts of the app, the translations are being fetched and the `NSFetchedResultsControllerDelegate` must respond to content changes.

```swift
extension ContentView {
    class ViewModel: NSObject, ObservableObject {

        @Published var translations: [Translation] = []

        let dataController: DataController
        private var translationsController: NSFetchedResultsController<Translation>

        init(dataController: DataController) {
            self.dataController = dataController
            let request: NSFetchRequest<Translation> = Translation.fetchRequest()
            request.sortDescriptors = []
            translationsController = NSFetchedResultsController(
                fetchRequest: request,
                managedObjectContext: dataController.container.viewContext,
                sectionNameKeyPath: nil,
                cacheName: nil
            )
            super.init()
            translationsController.delegate = self
            try? translationsController.performFetch()
            translations = translationsController.fetchedObjects ?? []
        }

    }
}

// MARK: - NSFetchedResultsControllerDelegate methods

extension ContentView.ViewModel: NSFetchedResultsControllerDelegate {
    func controllerDidChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
        if let newTranslations = controller.fetchedObjects as? [Translation] {
            translations = newTranslations
        }
    }
}
```

As the fetched translations are `@Published`, all the views that would like to update responsively to changes in translations, must bind and observe them with `onChange`.

```swift
struct SpreadsheetView: View {

    @Binding private var translations: [Translation]
    
    var body: some View {
        VStack {
            // ...
        }
        .onChange(of: translations) { newTranslations in
            viewModel.checkIfImported(translations: newTranslations)
        }
    }
}
```


---

