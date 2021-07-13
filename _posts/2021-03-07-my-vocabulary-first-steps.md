---
layout: post
title: First steps
subtitle: How to create a SwiftUI project with UIKit App Delegate life cycle?
comments: true
readtime: true
hidden: true
---

I created an empty SwiftUI project with UIKit App Delegate life cycle.

Then, I added `GoogleSignIn` and `GoogleAPIClientForREST/Drive` libraries to it, using CocoaPods.

To add CocoaPods to a project, run `pod init` in its main directory. If you don't have `pod` installed on your computer, check [this](https://cocoapods.org/).

The deployment target of my app is iOS 14.0, but your `Podfile` must look like mine.
```
platform :ios, '14.0'

target 'MyVocabulary' do
  use_frameworks!
  pod 'GoogleSignIn'
  pod 'GoogleAPIClientForREST/Drive'
end
```

These libraries are necessary because I will need to authenticate myself to access my Google Drive files.

Finish and run `pod install`.

Once you've installed CocoaPods, close your project, open Xcode again and select your **workspace**. This is very important, otherwise, your project won't build.

<img src="../assets/img/my-vocabulary/pods-xcode.png" class="center">


---

