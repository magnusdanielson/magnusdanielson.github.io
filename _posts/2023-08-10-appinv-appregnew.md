---
layout: post
title:  "Appinv and Appregnew missing in SharePoint online"
author: magnus
categories: [ SharePoint ]
image: assets/images/12.jpg
---
In SharePoint online the trusted appregnew.aspx and appinv.aspx has been disabled. The reenable them again
you can just run below powershell script.


```powershell
Connect-SPOService -URL https://dunite-yourtenant.sharepoint.com
Set-SPOTenant -SiteOwnerManageLegacyServicePrincipalEnabled $true
```
