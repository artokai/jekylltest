---
title: "Email address is not the user's identity in O365"
header:
  teaser: "images/posts/2016-06-07_teaser.jpg"
tags:
- O365
- SharePoint
---

Usually the user's identity in O365 is the same thing as their email address, but 
as a developer you should never assume that this is true for all users. 
Sometimes the environment can be configured in a way that the User Principal Name (UPN) is 
actually different than the user's primary email address.

Assuming that the user's UPN equals their email address can lead to subtle bugs which are 
hard to spot during testing. In some organizations there might be just a couple of users 
whose primary email address does not match their UPNs. It can be very tricky to find out 
why your custom code works over 99% of the time, but seems to fail for those few users.

One situation where this problem can occur is demonstrated in the following sample, which 
tries to create a new site collection and fails because John  is actually 
`john.smith@contoso.com` and not `john.smith@contoso.fi` which is just his primary email
address.

```cs
var tenant = new Tenant(ctx);
var props = new SiteCreationProperties()
{
    Url = "https://contoso.sharepoint.com/sites/testsite",
    Title = "TestSite",
    Template = "STS#0",
    StorageMaximumLevel = 1,
    UserCodeMaximumLevel = 0,
    Owner = "john.smith@contoso.fi",
};
var spoOperation = tenant.CreateSite(props);
ctx.Load(spoOperation);
ctx.ExecuteQuery();
```

It's easy to slip in these kind of errors in your code since often the required user information
is stored in a SharePoint list. The [FieldUserValue-class](https://msdn.microsoft.com/en-us/library/microsoft.sharepoint.client.fielduservalue.aspx)
and especially its [Email-property](https://msdn.microsoft.com/en-us/library/microsoft.sharepoint.client.fielduservalue.email.aspx) 
offer us easy access to the *wrong* information (which just happens to work 99% of the time and therefore the error goes unnoticed).
Unfortunately the FieldUserValue-class does not contain a direct property fors the user's login name, 
so we need to fetch it manually. One way to do this is to load it from the hidden 
[User Information List](https://gallery.technet.microsoft.com/User-Information-List-in-8b420e8c) stored in the root of the site collection.

```cs
var userFieldValue = (FieldUserValue) listItem["CustomUserField"];
var userItem = web.SiteUserInfoList.GetItemById(userFieldValue.LookupId);
ctx.Load(userItem);
ctx.ExecuteQuery();
var userName = userItem["UserName"];
```

If you are interested in finding out which of your users have a mismatch between their
UPNs and primary email addresses, you can list them using the following PowerShell script. The script
excludes all Guest-users since their UPN never matches their email-address anyway.

```powershell
$cred = Get-Credential
Connect-MsolService -Credential $cred
Get-MsolUser |? { $_.UserType -ne 'Guest' } | ForEach-Object {
    $user = $_;
    New-Object -TypeName PSObject -Property @{
        UserPrincipalName = $user.UserPrincipalName
        PrimaryEmail = (
                          $user | 
                          select -Expand ProxyAddresses |
                          ? {$_ -cmatch '^SMTP\:.*'}
                        ) -replace '^SMTP:', ''
    }
} | 
Where-Object { $_.PrimaryEmail -ne '' }  | 
Where-Object { $_.UserPrincipalName -ne $_.PrimaryEmail } 
```

**Note:** In order to run the script, you'll need to have the Windows Azure Active Directory Module installed on your system.
You can find the necessary installation and usage instructions in [this technet article](https://technet.microsoft.com/en-us/library/dn975125.aspx).
{: .notice--info}
