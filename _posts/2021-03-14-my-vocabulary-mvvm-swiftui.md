---
layout: post
title: MVVM in SwiftUI
subtitle: Is a good pattern in SwiftUI?
comments: true
readtime: true
hidden: true
---

To isolate the business logic from view, this is a useful pattern. The view will be reactive to an observable status in the view model.

### Import spreadsheets

The main view will contain the list of spreadsheets and the sign-in / out buttons. Each spreadsheet is represented by another view, that contains information about the spreadsheet and a button to import all the words from it.

Main view status:
```swift
enum Status {
    case off
    case loading
    case loaded(spreadsheets: [Spreadsheet])
}
```
Spreadsheet view status:
```swift
enum Status {
    case importable
    case imported
}
```
Status property must be inside a view model and each view will have its own, for example:
```swift
extension ImportView {
    class ViewModel: NSObject, ObservableObject {

        enum Status {
            case off
            case loading
            case loaded(spreadsheets: [Spreadsheet])
        }

        @Published private(set) var status: Status = .off
    }
}
```
To use the view model from the view:
```swift
struct ImportView: View {

    @StateObject private var viewModel: ViewModel

    init(viewModel: ViewModel) {
        _viewModel = StateObject(wrappedValue: viewModel)
    }
}
```

In summary:

- The view model must be a class that implements `ObservableObject`.
- The properties that will be observed, must be marked as `@Published`.
- The view model must be a `@StateObject` in the view.
- The view model must be initialized using the `StateObject(wrappedValue:)` initializer.

After doing this, the view model's status can be observed from the view:
```swift
var body: some View {
    NavigationView {
        if case .loading = viewModel.status {
            // ...
        } else if case .loaded(let spreadsheets) = viewModel.status {
            // ...
        } else {
            // ...
        }
    }
}
```

#### Bindings

After importing the words in a spreadsheet into the app, the fetched translations will be updated and each spreadsheet must indicate if its translations can be imported or if they have already been imported.

From the main view model, all the imported translations are fetched:
```swift
class ViewModel: NSObject, ObservableObject {

    // ...

    private let dataController: DataController
    private let translationsController: NSFetchedResultsController<Translation>

    @Published var translations: [Translation] = []

    // ...

    init(dataController: DataController, /* ... */) {
        self.dataController = dataController
        // ...
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
```
The view model becomes the delegate of the `NSFetchedResultsController`, receiving the the new fetched translations every time they are updated:
```swift
extension ImportView.ViewModel: NSFetchedResultsControllerDelegate {
    func controllerDidChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
        if let newTranslations = controller.fetchedObjects as? [Translation] {
            translations = newTranslations
        }
    }
}
```
The fetched translations are marked as `@Published`, so changes in the array will be notified to all its observers. From the main view, each time a spreadsheet view is created, the binding to the `@Published` translations is passed:
```swift
ForEach(spreadsheets, id: \.id) { spreadsheet in
    SpreadsheetView(
        translations: $viewModel.translations,
        viewModel: .init(spreadsheet: spreadsheet) {
            viewModel.importTranslations(from: spreadsheet)
        }
    )
    .padding()
}
```
To update the spreadsheet's view reactively with translation's changes, the published value must be bound:
```swift
struct SpreadsheetView: View {

    @Binding var translations: [Translation]

    // ...

    init(translations: Binding<[Translation]>, /* ... */) {
        self._translations = translations
        //...
    }
```
Every time the translations are updated, each spreadsheet checks if its translations have been imported. If this is the case, the status of the spreadsheet's view becomes `imported`.


---

