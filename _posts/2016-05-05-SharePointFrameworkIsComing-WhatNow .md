---
title: "The SharePoint Framework is coming, what now?"
header:
  teaser: "images/posts/2016-05-05-teaser.jpg"
tags:
- SharePoint
---

As you all probably know already, Microsoft announced their new 
page model for SharePoint yesterday. It seems that the new model 
changes significally the way we customize SharePoint in the future.
The tradional WebParts and AppParts will be replaced with new 
javascript based components which will utilize the new 
"SharePoint Framework". You can find some more information about the new framework in [this
blog post](https://blogs.office.com/2016/05/04/the-sharepoint-framework-an-open-and-connected-platform/) 
by Microsoft.

The SharePoint Framework or the new page model are not out yet 
and we need to keep on working and creating those old-school 
customizations for our current customers. At first I felt that this
can be a bit problematic since even though you can add those new 
SharePoint Framework based webparts on old SharePoint pages, 
you cannot add old fashnioned webparts or add-ins to those new
mobile friendly pages.

**Then I realized that the new model is not so new after all!** We've
been already writing javascript based "scriptparts" and used them
instead of traditional Webparts. Vesa Juvonen from Microsoft 
[blogged about this pattern in 2014](https://blogs.msdn.microsoft.com/vesku/2014/07/08/introducing-app-script-part-pattern-for-office365-app-model/) 
and the SharePoint Framework does not seem differ
too much from that.

<figure class="align-center">
  <img class="align-center" src="/images/posts/2016-05-05_AppScriptPart.jpg" style="max-width: 656px" alt="App Script Part Pattern"/>
  <figcaption style="text-align:center;">App Script Part Pattern by Vesa Juvonen</figcaption>
</figure>

In order to prepare for the new framework and make it easier to port
our existing solutions to the new platform, I think we should just 
keep on writing our scriptparts and make sure that we have 
a clean separation between the data access, business logic and 
the presentation layer. Using TypeScript to write these kind of scriptparts
is a good choise since it's module system makes it easy to separate 
these concerns into different modules with different responsibilities. 

<figure class="align-center">
  <img class="align-center" src="/images/posts/2016-05-05-the_3_tiers.jpg" style="max-width: 526px" alt="3-tier model"/>
  <figcaption style="text-align:center;">The three tiers in both ScriptParts and in the new SharePoint Framework model</figcaption>
</figure>

When the time comes, we can gradually port our data access layer to use 
the Microsoft Graph instead of the current SharePoint REST endpoints. 
And when we make the move to the new page model, we'll just have to port
our presentation layer to use the SharePoint Framework. In the best case 
scenario we don't even have to touch our business logic layer at all.

Of course porting existing code to the new framework will not happen 
automatically. But if we stick to the ScriptPart pattern and separate
our concerns carefully (as we always should anyway), we don't need to
throw away all of the code we write today. 