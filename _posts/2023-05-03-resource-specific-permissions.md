---
layout: post
title:  "Tree of Codes"
author: magnus
categories: [ Jekyll, tutorial ]
image: assets/images/9.jpg
---
As a Teams app developer, you may need to access various resources within a user's Teams environment, such as messages, files, or channels. However, you need to ensure that users are comfortable granting access to these resources and that their data is secure. That's where resource-specific consent comes in.

Resource-specific consent is a feature that allows Teams app developers to request access to specific resources within a user's Teams environment, such as messages or files. This means that users can grant access to only the resources that the app needs, rather than granting full access to their Teams environment. In this blog post, we'll walk you through how to implement resource-specific consent for your Teams app.

Step 1: Register your app

To use resource-specific consent, you must first register your app with the Microsoft Teams app store. Once your app is registered, you can configure it to request access to specific resources. To register your app, follow these steps:

Go to the Microsoft Teams Developer Portal (https://dev.teams.microsoft.com/).

Click on "Create an app" and enter your app's details.

Once your app is created, you'll be taken to the app's overview page. Here, you can configure the app's settings and permissions.

Step 2: Configure resource-specific consent

Once your app is registered, you can configure it to request access to specific resources within a user's Teams environment. To do this, follow these steps:

In the app's overview page, click on "Permissions."

Select the resource that your app needs access to. For example, if your app needs access to read messages in the channel, add ChannelMessage.Read.Group

Also open the manifest for the Teams app. Verify that `id` and `webApplicationInfo` is the same as

```json
 "id": "YOURCLIENTID",
 "webApplicationInfo": {
    "id": "YOURCLIENTID",
    "resource": "https://YOUR-HOST-URL"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        {
          "name": "TeamSettings.Read.Group",
          "type": "Application"
        },
        {
          "name": "ChannelMessage.Read.Group",
          "type": "Application"
        },
        {
          "name": "TeamSettings.Edit.Group",
          "type": "Application"
        }
      ]
    }
  }

Step 3: Request consent from users

Now that your app is configured to request access to specific resources, you can request consent from users. When a user installs your app, they will be prompted to grant permission to the specific resources that your app needs. That is all there is to it.

Step 4: Create a token

Things are now prepared for your backend code to create a token and then request information from Micorosft graph. Below function creates a token with the resource-specific permission for the application. Note that below does not work if you want to create a token with delegated permissions.

```c
public async Task<string> GetToken(string tenantId)
{
    IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(_configuration["ClientId"])
                                          .WithClientSecret(_configuration["ClientSecret"])
                                          .WithAuthority($"https://login.microsoftonline.com/{tenantId}")
                                          .WithRedirectUri("https://daemon")
                                          .Build();
    string[] scopes = new string[] { "https://graph.microsoft.com/.default" };
    var result = await app.AcquireTokenForClient(scopes).ExecuteAsync();
    return result.AccessToken;
}
```
Resource-specific consent is an essential feature for Teams app developers who want to ensure that users are comfortable granting access to their Teams environment. By requesting access to specific resources, you can build trust with users and ensure that their data is secure. Follow the steps outlined in this blog post to implement resource-specific consent for your Teams app and improve the user experience.

