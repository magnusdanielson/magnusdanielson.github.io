---
layout: post
title:  "Getting 401 when restoring from custom NuGet-feed"
author: magnus
categories: [ Nuget, 401 ]
image: assets/images/15.jpg
---
We have had a custom NuGet-feed in AzureDevops for a long time. Everytime we switch to a newer developer machine we need to restore access to the custom feed. The error we see is a plain Unauthorized.

```c
error :   Response status code does not indicate success: 401 (Unauthorized).
```

## Create a PAT
Follow below instructions to create a PAT for AzureDevops.

Log in to your Azure DevOps account and navigate to your profile settings.

From the sidebar menu, select "Security".

Under "Personal access tokens", click "New Token" to create a new PAT.

Choose a name for your token and select the organization you want it to apply to.

Select the desired scopes for your token. These scopes determine what actions your token can perform.

Set the expiration date for your token.

Click "Create Token" to generate your PAT.

After creating your PAT, you'll be able to see it in the "Personal access tokens" list. Make sure to copy and store the token somewhere safe, as it won't be displayed again.

## Download nuget
Go to the NuGet Downloads page (https://www.nuget.org/downloads) and download the latest version of nuget.exe. Save the file to a location on your computer that you can easily access. 

## Add the feed
First clear all old references of the feed from Visual Studio. Then add the feed with below command:

```
nuget.exe sources Add -Name "{yourfeedname}" -Source "https://pkgs.dev.azure.com/in3/_packaging/{yourfeedname}/nuget/v3/index.json" -username {youremail} -password {PAT token}
```

Restart Visual Studio so it picks up the new configuration. Now you should be good to restore nugets from your custom feed.