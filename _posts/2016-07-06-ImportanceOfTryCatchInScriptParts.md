---
title: "The importance of Try-Catch statements in SharePoint ScriptParts"
header:
  teaser: "images/posts/2016-07-06_teaser.jpg"
tags:
- SharePoint
- JavaScript
---

I received a report that one SharePoint [ScriptPart](https://blogs.msdn.microsoft.com/vesku/2014/07/08/introducing-app-script-part-pattern-for-office365-app-model/) 
was not functioning properly and spent quite a lot of time finding 
out the reason for it. It was a bit hard to find out the cause of the issue partly because
the issue concerned only IE and happened only when IE developer tools were not opened. 
This meant that I could not easily step through the code using IE's built-in debugger.

But the most time consuming thing was the error report itself. It turned out that the 
ScriptPart mentioned in the issue report was actually working just fine, but was just 
never executed. The SharePoint page in question had multiple ScriptParts and all the 
other ScriptParts *appeared* to be working just fine. But in the background one of those
other ScriptParts was throwing an unhandled exception... 

To demonstrate this behaviour, I created two simple ScriptParts and embedded them to a
SharePoint page using two separate Content Editor webparts. The first ScriptPart will throw an exception
since it tries to call a method that does not exist.

```javascript
<div id="scriptpart_one">Loading...</div>
<script type="text/javascript">
    (function() {
        SP.SOD.executeFunc('sp.js', 'SP.ClientContext', spReady);

        function spReady() {
            nonExistingFunction();
            var elem = document.getElementById("scriptpart_one");
            elem.innerText = "Hello from ScriptPart One!"
        }
    })();    
</script>
```

```javascript
<div id="scriptpart_two">Loading...</div>
<script type="text/javascript">
    (function() {
        SP.SOD.executeFunc('sp.js', 'SP.ClientContext', spReady);

        function spReady() {
            var elem = document.getElementById("scriptpart_two");
            elem.innerText = "Hello from ScriptPart Two!"
        }
    })();    
</script>
```

As you can see from the following screenshot, SharePoint will never 
execute the `spReady` callback of the second ScriptPart, because
the callback in the first ScriptPart does not handle the error properly.

<figure class="align-center">
  <img class="align-center" src="/images/posts/2016-07-06_trycatch.png" style="max-width: 720px" alt="Client WebPart Development Cycle"/>
</figure>

If the problematic ScriptPart would have a try-catch statement inside 
its executeFunc callback, it would not not prevent other registered 
callbacks from executing. A better version of the first ScriptPart is
presented below.

```javascript
<div id="scriptpart_one">Loading...</div>
<script type="text/javascript">
    (function() {
        SP.SOD.executeFunc('sp.js', 'SP.ClientContext', spReady);

        function spReady() {
            var elem = document.getElementById("scriptpart_one");
            try {
                nonExistingFunction();                
                elem.innerText = "Hello from ScriptPart One!"
            } catch (e) {
                displayError(e);
            }
        }

        function displayError(e) {
            var errorMsg = "Error: " + e.message;
            if (window.console && console.log)
                console.log(errorMsg);

            var elem = document.getElementById("scriptpart_one");
            if (elem)
                elem.innerText = errorMsg;                
        }
    })();    
</script>
```

(Note that you should not call ```console.log``` without first checking
that it really exists. Unchecked calls to ```console.log``` is one of those bugs 
which are hard to reproduce when IE's debugger is running.)
