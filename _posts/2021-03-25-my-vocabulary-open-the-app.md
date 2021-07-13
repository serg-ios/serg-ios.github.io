---
layout: post
title: Open the app
subtitle: With UIApplicationShortcutItem and NSUserActivity
comments: true
readtime: true
hidden: true
---

How to handle the quick action from the app? 

In the `SceneDelegate`.

If the app is in **background**.

```swift
func windowScene(
    _ windowScene: UIWindowScene,
    performActionFor shortcutItem: UIApplicationShortcutItem,
    completionHandler: @escaping (Bool) -> Void
) {
    // Intercept here the shortcutItem.type when the app is already launched but in background.
    contentViewModel.appAction = AppAction(shortcutItem.type)
}
```

If the app is completely **closed**.

```swift
func scene(
    _ scene: UIScene,
    willConnectTo session: UISceneSession,
    options connectionOptions: UIScene.ConnectionOptions
) {
    // Intercept here the connectionOptions.shortcutItem.type when the app is not launched.
    if let shortcutItem = connectionOptions.shortcutItem {
        appAction = AppAction(shortcutItem.type)
    } else if let userActivity = connectionOptions.userActivities.first {
        appAction = AppAction(
            userActivity.activityType,
            translation: translation(from: userActivity)
        )
    }
    // ...
}
```

There are other user activities that can launch the app. These can be Siri Shortcuts, Spotlight or Widgets.

That user activities can be intercepted as well in the previous function using `connectionOptions.userActivities.first`, check the previous block of code.

This function must be implemented:

```swift
func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    contentViewModel.appAction = AppAction(
        userActivity.activityType,
        translation: translation(from: userActivity)
    )
}
```


---

