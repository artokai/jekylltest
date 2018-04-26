---
title: "Did the UserName field just disappear from the User Information list in SharePoint Online?"
header:
  teaser: "images/posts/2016-08-19_teaser.jpg"
tags:
- SharePoint
---

One of our SharePoint AddIns just stopped working in SharePoint Online even though
we had done no changes in the actual codebase. It turned out that a field called ```UserName``` was
not present in the 
[User Information List](https://blogs.technet.microsoft.com/marj/2016/03/14/what-is-hidden-user-information-userinfo-list-in-sharepoint-20102013-and-how-to-fix-when-it-causes-a-site-collection-to-show-old-user-metadata-properties-in-people-picker-control-or-in-a-person-or/)
of a newly created site collection.

I checked a couple of newly created site collections and compared them with some older ones and
this out-of-the-box field is indeed no longer present in the new site collections. And to make things worse, 
this field is missing only in our production tenant. When I create new site collections in our development tenant 
the field is still present and our code works just fine.

I can think of two possible explanations for this missing field and I really don't like either one of them:

1. There is a temporary problem with our production tenant (unlikely, since everything else seems to work just fine)
2. Microsoft just decided to remove this field from the list without telling us about it. 

Just a few weeks ago Microsoft broke their customers' solutions by 
[disabling sandboxed solutions](http://www.sharepointnutsandbolts.com/2016/08/sandbox-code-disabled-in-Office-365.html) 
without bothering to inform them properly about it. And now they seem to have made another change which breaks things up. 
This time the change is a smaller one, but it still took us by surprise. I really hope that Microsoft will figure out some way
to keep us informed about these things. 

So if you're facing issues with the User Information List, you can check if the UserName field is present in your tenant with the following PowerShell Script. 
In order to run it, you need to have the magnificent [PnP PowerShell Cmdlets](https://github.com/OfficeDev/PnP-PowerShell) installed on your system. 

```powershell
$creds = Get-Credential
$siteUrl = "https://tenantname.sharepoint.com/sites/your_site_name"
Connect-SPOnline -Url $siteUrl -Credentials $creds
$web = Get-SPOWeb
$ctx = Get-SPOContext
$siteUserInfoList = $web.SiteUserInfoList;
$ctx.Load($siteUserInfoList)
$ctx.Load($siteUserInfoList.Fields)
$ctx.ExecuteQuery()
$siteUserInfoList.Fields | Where-Object { $_.InternalName -eq "UserName"}  
Disconnect-SPOnline
```
