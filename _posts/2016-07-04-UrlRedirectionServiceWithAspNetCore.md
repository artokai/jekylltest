---
title: "Url redirection service with ASP.NET Core"
header:
  teaser: "images/posts/2016-07-04_teaser.jpg"
tags:
- ASP.NET
- Azure
---

I needed a personal URL redirection service to make it easier to publish 
more memorable url addresses. For example, if you want to see my LinkedIn profile 
you can now use the url [http://aka.artokai.net/linkedin](http://aka.artokai.net/linkedin) 
and you will be redirected to my linked in profile. 

Actually I didn't really *NEED* a url redirection service, but I wanted to have one.
And that's almost the same thing right? So I did a quick google search for available 
options and did not really like what I found. I wanted to host my redirection service 
in Azure and I wanted to keep the costs as low as possible. 
This meant that I did not want to pay for a database for example.

So when [ASP.NET core 1.0](http://www.asp.net/core) was released I decided to build
my own url redirection service with only a minimal set of features and functionalities. 
The service, which I named Azurl, holds all url aliases in memory and therefore does 
not require a backing database server. To survive reboots the aliases are also
serialized to disk in JSON format.

Azurl is just a url redirection service. It does not contain any user interfaces at all. 
All maintenance work (adding and removing aliases) are done through 
a [JSON file stored in a GitHub repository](https://github.com/artokai/Azurl/blob/master/aliases.json). 
When this file is changed, Azurl receives a notification about it through a webhook registered in GitHub. 

<figure class="align-center">
  <img class="align-center" src="/images/posts/2016-07-04_Azurl_Sequence.png" style="max-width: 720px" alt="Client WebPart Development Cycle"/>
  <figcaption style="text-align:center;">Azurl configuration process</figcaption>  
</figure>

In the end I'm quite happy with the results. Editing the alias-configuration can be done directly
from the GitHub UI and the new aliases are available almost immediately. Working with the new 
ASP.NET Core was also a nice experience. I really liked the new built-in dependency injection
framework and configuration management model.

The service and sources can be found in [http://aka.artokai.net/azurl](http://aka.artokai.net/azurl) =) 
