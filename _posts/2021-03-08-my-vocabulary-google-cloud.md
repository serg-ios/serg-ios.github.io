---
layout: post
title: Google Cloud Platform
subtitle: How to obtain an API key?
comments: true
readtime: true
hidden: true
---

The first thing that must be done is authentication with Google credentials to access Google Drive, a **Google API Key** it's necessary .

1️⃣ Visit your Google Cloud Platform [dashboard](https://console.cloud.google.com/projectselector2/home/dashboard) and create a project.

2️⃣ Give a name to it.

<img src="../assets/img/my-vocabulary/google-cloud/google_project_name.jpg" width="300" class="center">

3️⃣ Go to your APIs.

<img src="../assets/img/my-vocabulary/google-cloud/see-apis.png" width="300" class="center">

4️⃣ Enable Google Drive API.

<img src="../assets/img/my-vocabulary/google-cloud/enable-drive.png" width="300" class="center">

5️⃣ Create credentials.

<img src="../assets/img/my-vocabulary/google-cloud/create-credentials.png" width="300" class="center">

6️⃣ Configure OAuth screen.

<img src="../assets/img/my-vocabulary/google-cloud/configure-oauth2.png" width="300" class="center">

7️⃣ Add testing emails, here you should add the Google account in which you have your translations. You will not be able to sign in during development with no registered testing emails.

<img src="../assets/img/my-vocabulary/google-cloud/oauth-email.png" width="400" class="center">

8️⃣ Finish the configuration of your OAuth client.

<img src="../assets/img/my-vocabulary/google-cloud/finish.png" width="300" class="center">

9️⃣ In APIs and Services > Credentials, download the plist that contains `CLIENT_ID` and `REVERSED_CLIENT_ID`.

<img src="../assets/img/my-vocabulary/google-cloud/download.png" class="center">


---

