---
title: "Testing with Chuck"
header:
  teaser: "images/posts/2016-04-28_chuck_teaser.jpg"
tags:
- SharePoint
- PowerShell
---

Sometimes you need to populate your SharePoint list with test data in order to test how your custom
functionality works when there are more then four items added to the list. Of course it's easy to 
add the required test data using PowerShell and populate the list with like ```test item #1```, 
```test item #2```, ```test item #3```, and so on...

However looking at that kind of test items gets boring quite fast. To fix this we can utilize the 
magnificient [The Internet Chuck Norris Database](http://www.icndb.com/) to load a lot more interesting 
test values in our list.

<figure class="align-center">
  <img class="align-center" src="/images/posts/2016-04-28_ChuckNorrisCartoon.jpg" style="max-width: 466px" alt="Chuck and SharePoint"/>
  <figcaption style="text-align:center;">Chuck Norris and SharePoint cartoon by <a href="https://blogs.technet.microsoft.com/sharepointcomic">Dan Lewis</a></figcaption>
</figure>

First you need to install the [OfficeDevPnP.PowerShell Commands](https://github.com/OfficeDev/PnP-PowerShell) 
on your computer unless you've already done that. These CmdLets make writing SharePoint related scripts so much
easier and even Chuck Norris can't live without them. Then start up PowerShell and run the following script
to add Chuck related testdata to your SharePoint list.  

``` powershell
$siteUrl = "https://contoso.sharepoint.com/sites/sitename"
$listname = "ChuckFacts"
$icndbUrl = "http://api.icndb.com/jokes"

$credential = Get-Credential
Connect-SPOnline -Url $siteUrl -Credentials $credential
Invoke-RestMethod -Uri $icndbUrl | Select -ExpandProperty value | ForEach-Object {
    $fact = [system.web.httputility]::htmldecode($_.joke).Trim();
    if ($fact.Length -lt 256) {
        Add-SPOListItem -List $listname -Values @{"Title" = $fact }
    }
}
Disconnect-SPOnline
```

Once again Chuck Norris has saved our day and the world is a safe place again. Happy testing!

