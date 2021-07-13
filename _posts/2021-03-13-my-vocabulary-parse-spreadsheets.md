---
layout: post
title: Parse spreadsheet's translations
subtitle: Using CoreXLSX
comments: true
readtime: true
hidden: true
---

This is not as straightforward as fetching the data, to do this, there are libraries like [CoreXLSX](https://github.com/CoreOffice/CoreXLSX).

Adding it to your project is very simple with CocoaPods.
```
platform :ios, '14.0'

target 'MyVocabulary' do
  use_frameworks!
  # ...
  pod 'CoreXLSX'
end
```

### ♦️ Important

Before doing `pod install`, mark the `xcconfig` files as **no-assume-unchanged**.
```
git update-index --no-assume-unchanged MyVocabulary/Pods/Target\ Support\ Files/Pods-MyVocabulary/Pods-MyVocabulary.release.xcconfig
git update-index --no-assume-unchanged MyVocabulary/Pods/Target\ Support\ Files/Pods-MyVocabulary/Pods-MyVocabulary.debug.xcconfig
```

Then, install the new pod and you'll see that there are changes in that 2 files, check the files before uploading the changes (if you don't want to upload your API keys) and then upload them.

Now, the keys can be added again to the `xcconfig` files and the files must be marked again as **assume-unchanged**.
```
git update-index --assume-unchanged MyVocabulary/Pods/Target\ Support\ Files/Pods-MyVocabulary/Pods-MyVocabulary.debug.xcconfig
git update-index --assume-unchanged MyVocabulary/Pods/Target\ Support\ Files/Pods-MyVocabulary/Pods-MyVocabulary.release.xcconfig
```

To avoid uploading API Keys to your repo, do this every time you want to add a new library via CocoaPods.

### ♦️

Once installed the framework, here is how the rows of the spreadsheet can be accessed. Each row is an array of cells, this operation can throw several kinds of errors that can be handled.
```swift
import CoreXLSX
// ...
do {
    let file = try XLSXFile(data: data)
    guard let sharedStrings = try file.parseSharedStrings() else { return }
    let workbooks = try file.parseWorkbooks()
    for workbook in workbooks {
        let pathsAndNames = try file.parseWorksheetPathsAndNames(workbook: workbook)
        for (_, path) in pathsAndNames {
            let worksheet = try file.parseWorksheet(at: path)
            for row in worksheet.data?.rows ?? [] {
                let originLanguage = row[0]
                let aimLanguage = row[1]
                let originalText = row[2]
                let resultText = row[3]
                // ...
            }
        }
    }
} catch {
    // ...
}
```

And that's it! With this, you can read a spreadsheet from your Google Drive and use the data in your app.

---

