---
title: "Office Online integrations revisited"
header:
  teaser: "images/posts/2016-04-26_teaser.jpg"
tags:
- O365
- Office Online
---

When I first started blogging about three years ago, I started to write 
a blog post series about using Office Web Apps to view and edit
files stored in your custom applications. However back then it was way too tedious 
to allow your users to edit MS Word documents, so I silently cursed Microsoft 
and finally gave up.

> As you may already have noticed, my blog post series about Office Web Apps 2013 Integration has been terminated. The idea about sending SOAP-messages inside a RESTful API is just plain wrong and I got tired of it.
>
> <cite>The termination notice of my original blog post series</cite>

The main culprit was a protocol called [MS-FSSHTTP](https://msdn.microsoft.com/en-us/library/dd943623(v=office.12).aspx) 
also known as "Cobalt". Cobalt makes it possible for multiple 
people to edit the same document at the same time by sending only 
fragments of the whole document to the server hosting the actual
document.

Unfortunately Cobalt was the *ONLY* allowed protocol to save 
MS Word -files and for simple document authoring purposes it was
way too complicated to implement. For example MS Excel files were
saved using the much simpler [MS WOPI](https://msdn.microsoft.com/en-us/library/hh622722(v=office.12).aspx)-protocol. 

But things seem to have changed. Back in 2013 information and 
especially examples regarding the protocols in question were somewhat scarce. 
And now Microsoft is inviting people to integrate their own 
applications to Office Online and has set up 
[very informative integration instructions](https://wopi.readthedocs.org) 
which contain for example a sample implementation and even a test suite which you can 
utilize when implementing your own integration to Office Online.

And they even added the following notice almost to the beginning of 
their instructions! It seems that I wasn't the only one who got
frustrated with MS-FSSHTTP.

> you don’t have to implement the [MS-FSSHTTP]: File Synchronization via SOAP over HTTP Protocol (Cobalt).
>
> <cite><a href="https://wopi.readthedocs.org/en/latest/overview.html#integration-process">https://wopi.readthedocs.org</a></cite>

Since the documentation and examples provided by Microsoft are so
good, there's really no point in resurrecting my earlier blog post series. 
But when I have more time (which I never seem to have), I will definitely
take a look at their sample application in more detail and see how it compares
to my earlier implementation attempt.
