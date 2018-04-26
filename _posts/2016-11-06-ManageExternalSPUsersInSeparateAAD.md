---
title: "Managing SharePoint external users in a separate Azure AD instance"
header:
  teaser: "images/posts/2016-11-06_teaser.jpg"
tags:
- SharePoint
- O365
- Azure
- Azure AD
---

In [my last blog post](/2016/SPOnlineAndAzureB2B/) I talked about how you can
utilize [Azure AD B2B](https://azure.microsoft.com/en-us/documentation/articles/active-directory-b2b-collaboration-overview/) 
to invite external users to your SharePoint sites. In this blog post I'll describe an
alternative method which you can use, if you want to manage the external user accounts 
in your own Azure Active Directory. 

The idea is to create a separate Azure AD for your external users and use that to manage 
and maintain the identities of your external users. Toni Pohl
has already [blogged about this approach](http://blog.atwork.at/post/2014/12/06/How-to-use-external-users-in-SharePoint-Online-with-a-cost-free-Azure-Active-Directory) 
back in 2014. His solution was to create a user account in a separate Azure AD,
send a SharePoint sharing invitation to the user's real email address and hope that
the user will sign in using the user account created in the separate Azure AD.

The method I'm going to describe next does not require you to send any
sharing invites from SharePoint and thus makes it impossible for the end user 
to accept the invite with some other user account than the one prepared for him. 
But this currently comes with a cost. It will leave the email address field in
the external user's user profile empty and therefore SharePoint will not be 
able to send any email notifications to the user.

Consider yourself warned!

### Step 1: Create a separate Azure AD to host your external users

First we need to create a new Azure AD instance which will contain our external users.
So login to [https://manage.windowsazure.com](https://manage.windowsazure.com), select "Active Directory" and press "New"
to create a new Azure AD instance. Creating a new Azure AD instance is pretty straight forward. 
You basically just need to provide a name and a location for the new directory and you're all set. 

### Step 2: Associate a domain name with your new AD (Optional)

By default all user's in your new directory will have usernames like 
"john.smith@contosocustomers.onmicrosoft.com". In business scenarios you probably want to use
some other domain than "onmicrosoft.com". If you want to change that to something like
"contosocustomers.com" you probably should do that now, before creating any actual users.

To add a new domain to your directory, you need to select the "Domains" tab of
your newly created domain and press the "Add a custom domain"-link. This will open up
a wizard that will guide you through the rest of the process. Just remember 
that adding a new domain requires you to prove that you actually manage the domain you're adding, 
so some DNS changes are required.

### Step 3: Add your users external users in the new AD

To add a new user in your new Active Directory, navigate to your newly created
Active Directory, select Users and press the "Add User"-button.

Select "New user in your organization" as the user's type and follow the
instructions provided by the wizard to create a user account for your external user.
When you're presented with a temporary password, write it down since you'll need to send it to 
your external user later on. 

### Step 4: Add your external users as guests in your primary AD

SharePoint allows you to grant permissions to only users in the primary Azure AD
associated with your subscription. So even though you have now created a user account for your
external user, you'll now need to add that account as a "guest" in your primary directory. 
This is actually something that the SharePoint's invitation process will do behind the
scenes automatically when a user accepts an invite to a SharePoint site.

**Note:** The next step needs to be done through the classic portal, since currently the
new portal does not support adding users from another directories.
{: .notice--info}

To create the required "guest-proxy", navigate to your primary directory and press "Add user" 
button in the Users-tab. You need to select "User in another Microsoft Azure AD directory"
as the user type and fill in the user name -field. As user's role we would like to fill 
in "Guest", but this is unfortunately not possible through the UI. So let's leave it to
"User" for now. We'll soon fix it with PowerShell.

<figure class="align-center">
  <img class="align-center" src="/images/posts/2016-11-06_AddUserToPrimaryDirectory.jpg" style="max-width: 770px" alt="Add user from another Azure AD"/>
</figure>

Now our external user is added to the primary domain and can be found by SharePoint Online.
However the user's role is still "User" and SharePoint does not like this. If the 
user would now try to login to SharePoint Online using his new user account, he would receive an error message saying
"We're sorry, but john.smith@contosocustomers.com can't be found in the contoso.sharepoint.com directory.".

To fix this we will need to change the user's role to "Guest" using the following 
PowerShell script:

```powershell
$cred = get-credential
Connect-MsolService -Credential $cred
$user = Get-MsolUser | Where-Object { $_.SignInName -eq "john.smith@contosocustomers.com"}
Set-MsolUser -ObjectId $user.ObjectId -UserType Guest
``` 

**Note:** At this point we would also like  to set user's ProxyAddresses field to include the 
user's real email address. This is the field that SharePoint will use for the user's email address.  
However Azure AD does not currently support this. The only way to set the ProxyAddresses-field is through
Exchange Online and it cannot be done to users without actual mailboxes.
{: .notice--info}

### Step 5: Grant permissions in SharePoint

After the guest user account is created in the directory associated with your O365 subscription,
you finally go to your SharePoint site and grant the necessary permissions to your external users.
When granting the permissions, you should use the newly created user account (for example 
"john.smith@contosocustomers.com") instead of the user's real email address. Since we already
created a "guest-proxy" for that user account, SharePoint will find this user and no sharing
invitations will be emailed to anyone.

**Note:** Since you are granting permissions to an external user, you need to allow external
sharing for the site collection before you try to grant permissions to the user.
{: .notice--info}

### Step 6: Send the login information to your external user

The final step is to contact your external user and provide him with the username, password
and the url of your shared site. If you followed these instructions correctly, he/she should be 
able to login to your SharePoint site without any problems.

## Conclusion and final thoughts

It is indeed possible to manage external user accounts in a separate Azure AD and grant permissions 
to those user accounts without emailing any sharing invitations to your users. But not having
the user's real email address populated in SharePoint is a major drawback of this method.

The process also includes quite a lot of steps for creating a single user. I would like
to write a single PowerShell script that would automate the whole process, but 
it seems that importing users from another Azure AD cannot be done programmatically.

So I'm not fully satisfied with this process and I suspect that the whole invite process is 
so strongly built-in to Azure AD, that resistance is futile. Perhaps I'll just take a closer
look on what's cooking inside the MS Graph API and check if the new 
[Invitation Manager-endpoint](https://graph.microsoft.io/en-us/docs/api-reference/beta/resources/invitation) 
(currently in beta) could somehow be used to make the SharePoint external user management easier...
