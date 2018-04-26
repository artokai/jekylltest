---
title: "Using Azure B2B to invite external users to SharePoint Online"
header:
  teaser: "images/posts/2016-10-13_teaser.jpg"
tags:
- SharePoint
- O365
- Azure
- Azure AD
---

**Note:** This post is related to Azure AD B2B, which is still in preview. This means that things may still change
before the final version Azure AD B2B is released.
{: .notice--info}

[Azure AD B2B](https://azure.microsoft.com/en-us/documentation/articles/active-directory-b2b-collaboration-overview/)
allows you to invite users from partner companies and let them access your
resources using partner-managed identities. Using a partner-managed identity means
that you don't have to create or maintain the user accounts for your external users.
The invited users can login to your SharePoint site using his/her own corporate user account,
which is a lot better than having them using their personal Microsoft accounts.

If your partner does not have an Azure AD tenant, then B2B will
notice this and automatically create a new Azure AD tenant for them based on the domain
name of the invited email address. These tenants are called "viral tenants".

At this point the tenant and the automatically created
user accounts ("viral users") within are not owned or maintained by any specific organization. Users
can reset their own passwords and so on, but no-one is assigned as an administrator
for the viral tenant. Of course your partner can later on claim the ownership of
the viral tenant by registering the domain name in question in their own subscription.
After that it will become just a regular Azure AD tenant.

<figure class="align-center">
  <img class="align-center" src="/images/posts/2016-10-13_b2bflow.jpg" style="max-width: 500px" alt="B2B invitation process"/>
</figure>

When using Azure AD B2B to invite users in partner companies, the invitation process will
also create a corresponding Guest user in your own Azure AD tenant. The created guest
account is similar to the one that gets created automatically when you share a SharePoint site
to an external user. The guest account created by B2B is actually created before the user
has even accepted the invitation, which means that you don't have to wait for the user to
accept the invitation before granting him/her permissios to your SharePoint site.

## How to invite users with Azure AD B2B?

Currently the Azure AD B2B invitation process starts with a CSV-file, which contains a
list of email addresses you want to invite as guests. This CSV file is then
uploaded to Azure AD admin portal, which starts the invitation process.

So the first step of inviting people is to create a CSV file for example using
MS Excel:

```csv
Email,DisplayName,InvitationText,InviteRedirectUrl,InvitedToApplications,InvitedToGroups,CcEmailAddress,Language
john@contoso.com,John Smith,,https://contoso.sharepoint.com/sites/b2bsite,,,,en
walter@contoso.com,Walter Harp,,https://contoso.sharepoint.com/sites/b2bsite,,,,en
ben@contoso.com,Ben Smith,,https://contoso.sharepoint.com/sites/b2bsite,,,,en
```

The format of the CSV-file is described [here](https://azure.microsoft.com/en-us/documentation/articles/active-directory-b2b-references-csv-file-format/)
and only the fields `Email` and `DisplayName` are actually required for the invitation process.
Note that you should always leave the `CcEmailAddress` field empty.
If this field is used, the invitation cannot be used for viral user or tenant creation.

**Note:**
If you live outside the US, you should be aware that when the B2B documentation talks about CSV it actually means
__comma-separated-values__. This means that editing this file in Excel may be a bit difficult,
if your CSV-files are usually separated by some other character. For example here in Finland we (and our
MS Excels) usually use semicolon (;) instead of commas (,) in our CSV files.
{: .notice--info}

To have your CSV-file processed, you need to login to Azure AD admin portal,
click "Add User" and select the type of the user to be `Users in parner companies`.
A detailed walkthrouh (with screenshots) of this can be found [here](https://azure.microsoft.com/en-us/documentation/articles/active-directory-b2b-detailed-walkthrough/).

<figure class="align-center">
  <img class="align-center" src="/images/posts/2016-10-13_b2b_upload_csv.jpg" style="max-width: 781px" alt="B2B invitation process"/>
</figure>

**Note:** Currently Azure AD B2B functionalities are available only in the Azure AD classic portal.
So in order to have your CSV file processed, you need to login to <a href="https://manage.windowsazure.com">https://manage.windowsazure.com</a>
and access your Active Directory from there.
{: .notice--info}

## Granting permissions to SharePoint

After the CSV file has been processed, the invited users will immediately appear
as guests in your own Azure AD and you can start granting permissions to them in
SharePoint. Adding a lot of users for example to site's Visitors-group can however require
quite a lot of manual work. You can automate this process by either granting
permissions to the site by using an Azure AD group or by using PowerShell.

### Option 1: Granting permissions through Azure AD groups

If you create an Azure AD group for your invited users, you can define its
group id in the `InvitedToGroups` field of the CSV. This will instruct B2B
to automatically add your invited users to that group. If you can grant permissions
to this Azure AD group in SharePoint, then all of your invited users will also
gain access to your SharePoint site.

Microsoft has published [instructions](https://azure.microsoft.com/en-us/documentation/articles/active-directory-b2b-detailed-walkthrough/#adding-carol-to-the-contoso-directory-granting-access-to-apps-and-giving-group-membership)
on how to automatically add invited users to specified AD groups. If you take
this route, be aware that you can also find the group's Object ID in the Azure Portal and
you don't necessary need to user PowerShell to look it up (as stated in the Microsoft's
instructions).

<figure class="align-center">
  <img class="align-center" src="/images/posts/2016-10-13_group_objectid.jpg" style="max-width: 720px" alt="Group's Object Id"/>
</figure>

### Option 2: Granting permissions directly to SharePoint using PowerShell

If you don't want to start managing SharePoint permissions through AD groups, you can
add your invited users directly to SharePoint groups. Luckily we can automate this step
with a little PowerShell script.

Since we already have a list of users defined in a CSV file, we can use the same CSV file
to grant permissions to our SharePoint site. We just need to add one
additional field (`SPGroups`) to the CSV file in order for our script to know to which
SharePoint groups the user should be added. I've chosen to separate multiple group names
with a plus sign (+).

```csv
Email,DisplayName,InvitationText,InviteRedirectUrl,InvitedToApplications,InvitedToGroups,CcEmailAddress,Language,SPGroups
john@contoso.com,Jeff Smith,,https://contoso.sharepoint.com/sites/b2bsite,,,,en,"B2Bsite Visitors
walter@contoso.com,Walter Harp,,https://contoso.sharepoint.com/sites/b2bsite,,,,en,"B2Bsite Visitors"
ben@contoso.com,Ben Smith,,https://contoso.sharepoint.com/sites/b2bsite,,,,en,"B2Bsite Members+B2Bsite Reviewers"
```

The [CSV format reference](https://azure.microsoft.com/en-us/documentation/articles/active-directory-b2b-references-csv-file-format/)
for B2B does not say anything about additional fields in the CSV-file,
but they do not seem to break the invitation process in any way. The B2B process just
ignores them which is great from our point of view since we can control both B2B and our PowerShell script
with exactly the same file.

After the extended CSV file has been processed by Azure AD B2B you can add the invited 
users to SharePoint groups by executing [a little script](https://gist.github.com/artokai/6c79ec4169eb35ba111c80aac0b5ec81)
in PowerShell. In order for the script to work, you need to have the [PnP PowerShell Cmdlets](https://github.com/OfficeDev/PnP-PowerShell) installed on your system.

```powershell
.\Add-B2BUsersToSite.ps1 -Verbose `
                         -CSVFilename c:\b2b_users.csv `
                         -SiteUrl https://contoso.sharepoint.com/sites/b2bsite


VERBOSE: Credentials not supplied, prompting for them.
Adding john@contoso.com to site
VERBOSE: Adding user to groups
VERBOSE: - Adding user to group: B2Bsite Visitors
Adding walter@contoso.com to site
VERBOSE: Adding user to groups
VERBOSE: - Adding user to group: B2Bsite Visitors
Adding ben@contoso.com to site
VERBOSE: Adding user to groups
VERBOSE: - Adding user to group: B2Bsite Members
VERBOSE: - Adding user to group: B2Bsite Reviewers
```
