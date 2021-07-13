---
layout: post
title: Download spreadsheets from Google Drive
subtitle: Using Google Drive API
comments: true
readtime: true
hidden: true
---

A **scope** must be added to the Google Sign In configuration in the App Delegate's `didFinishLaunchingWithOptions` method, to request the user permissions for accessing Google Drive files.

```swift
import GoogleAPIClientForREST
// ...
GIDSignIn.sharedInstance().scopes = [kGTLRAuthScopeDriveReadonly]
```

[Here](https://developers.google.com/drive/api/v3/reference/files/list) is the Google documentation for the Google Drive API requests that are needed.

A reference of the Google Drive service authorizer must be saved, add to the `GoogleSignInDelegate` a reference to the service.
```swift
private let googleDriveService = GTLRDriveService()
```
Then, in the implementation of `didSignIn`, save the authorizer.
```swift
googleDriveService.authorizer = user.authentication.fetcherAuthorizer()
```

### Fetch all spreadsheets

A `GTLRDrive_FileList` will be downloaded using Google Drive's REST API query. This can also download files from Google Photos, for instance. In this case, only Drive files are needed, so the `spaces` and `corpora` properties must be set.
```swift
let query = GTLRDriveQuery_FilesList.query()
query.spaces = "drive"
query.corpora = "user"
```
<img src="../assets/img/my-vocabulary/corpora.png" width="400" class="center"> 

<img src="../assets/img/my-vocabulary/spaces.png" width="400" class="center">

By default, not all the fields of the `GTLRDrive_File` are fetched, indicate which ones must be requested.
```swift
query.fields = "files(id,name,modifiedTime,size)"
```
Also, tell the query to look only for spreadsheets.
```swift
let spreadsheetOnly = "mimeType = 'application/vnd.google-apps.spreadsheet'"
query.q = "\(spreadsheetOnly)"
```
Then, just execute the query and wait for the result, a `GTLRDrive_FileList`.
```swift
googleDriveService.executeQuery(query) { [weak self] _, result, error in
    guard error == nil else { return }
    for file in (result as? GTLRDrive_FileList)?.files ?? [] {
        // ...
    }
}
```
This query will return all the spreadsheets linked with that account, even those that are shared with other users or those that are in the Trash. Filters can be added, for instance, to fetch only files owned by the user and not other user's shared files.
```swift
let ownedByUser = "'\(email)' in owners"
query.q = "\(spreadsheetOnly) and \(ownedByUser)"
```
### Fetch the content of each spreadsheet

The previous query only fetched info about the files, to get the whole file, other requests must be made with the Google Drive service fetcher.

The ID of each spreadsheet must be included in the URL, in following example, replace the %@ with it.
```swift
private func spreadsheetUrl(id: String) -> URL? {
    URL(string: String(format: "https://www.googleapis.com/drive/v3/files/%@/export?alt=media&mimeType=application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", id))
}
```
Use the URL to fetch the resource with Google Service.
```swift
googleDriveService.fetcherService.fetcher(with: url).beginFetch { [weak self] data, error in
    guard error == nil, let data = data else { return }
    // Handle the data
}
```


---

