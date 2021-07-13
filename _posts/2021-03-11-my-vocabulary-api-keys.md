---
layout: post
title: Don't upload API Keys to Git
subtitle: Or any other sensitive data
comments: true
readtime: true
hidden: true
---

If you have a public repository you might want to hide your Google Sign-In Client ID.

Instead of hardcoding API keys into `Info.plist` or `AppDelegate`, they can be hidden in many ways, so they are not available in your repo.

Open the `xcconfig` files of your project.

<img src="../assets/img/my-vocabulary/xcconfig.png" width="400" class="center">

```
MyVocabulary/Pods/Target\ Support\ Files/Pods-MyVocabulary/Pods-MyVocabulary.release.xcconfig
MyVocabulary/Pods/Target\ Support\ Files/Pods-MyVocabulary/Pods-MyVocabulary.debug.xcconfig
```

Append `REVERSED_CLIENT_ID` and `CLIENT_ID` at the end of both files (the name of the variables can be modified).

```
GOOGLE_SIGN_IN_REVERSED_CLIENT_ID=com.googleusercontent.apps.xxxxxxxxxxxxx
GOOGLE_SIGN_IN_CLIENT_ID=xxxxxxxxxxxxx.apps.googleusercontent.com
```

You don't want to upload these changes to your repo, so before doing `git add` of those files, you must indicate that changes in those files must not be taken into consideration.

```
git update-index --assume-unchanged MyVocabulary/Pods/Target\ Support\ Files/Pods-MyVocabulary/Pods-MyVocabulary.debug.xcconfig
git update-index --assume-unchanged MyVocabulary/Pods/Target\ Support\ Files/Pods-MyVocabulary/Pods-MyVocabulary.release.xcconfig
```

Note: If later you want to make changes into `xcconfig` files and upload them into your repo, remember to revert this.

```
git update-index --no-assume-unchanged <file>
```


---


Now, the variables can be used in the `Info.plist` doing `$(GOOGLE_SIGN_IN_CLIENT_ID)` and `$(GOOGLE_SIGN_IN_REVERSED_CLIENT_ID)`. Please, when adding the new URLTypes, give them an unique identifier.

<img src="../assets/img/my-vocabulary/urltypes.png" width="400" class="center">

We are almost there... Remember that the client ID was needed in the `AppDelegate` to be set into `GIDSignIn.sharedInstance().clientID`? I made an extension of `Bundle` that obtains URLType's URL Schemes giving its identifier (see the previous step).

```swift
extension Bundle {
    static func urlScheme(urlTypeId: String) -> String? {
        (((main.infoDictionary?["CFBundleURLTypes"] as? [[String: Any]])?
            .first(where: { $0["CFBundleURLName"] as? String == urlTypeId}))?["CFBundleURLSchemes"] as? [String])?.first
    }
}
```

Instead of hardcoding the API key in the `AppDelegate`...

```swift
GIDSignIn.sharedInstance().clientID = Bundle.urlScheme(urlTypeId: "GOOGLE_SIGN_IN_CLIENT_ID")
```


---

