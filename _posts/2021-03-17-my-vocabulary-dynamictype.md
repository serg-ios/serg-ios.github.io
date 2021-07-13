---
layout: post
title: Dynamic Type
subtitle: Make text bigger (or smaller)
comments: true
readtime: true
hidden: true
---

Some people need bigger fonts to be able to read the texts on the screen, there are a lot of apps out there that don't support the **Dynamic Type** feature.

For texts in SwiftUI, the simplest way of doing it is applying the view modifier `font` with any of the default fonts:

```swift
// ...
.font(.title2)
.font(.title3)
.font(.footnote)
.font(.largeTitle)
// ...
```

These are the ones I used, but there are many [more](https://developer.apple.com/documentation/swiftui/font).

<img src="../assets/img/my-vocabulary/accessibility/dynamic_types.gif" width="280" class="center">

When the font increases its size, some texts need more than one line to be represented, to limit the number of lines, use the view modifier `.lineLimit(1)`.

For those texts that must not be truncated, `.minimumScaleFactor(0.1)` can be used, so the font size scales accordingly to the factor if it doesn't fit in the available space.

But dynamic sizes don't apply exclusively to texts, images, and other views can become bigger too, according to the user settings:

- By giving them a fixed size, they won't grow nor shrink. 
- By using a minimum size and no maximum, for example, views can grow indefinitely and shrink down to a limit, just use the view modifier `.frame(minWidth: 44, minHeight: 44)`.

<img src="../assets/img/my-vocabulary/accessibility/xcode_dynamic_types.gif" width="280" class="center">

The size of the texts and views can be changed in runtime from Xcode.

---

