---
layout: post
title: "Workflow error - Cannot make a cache safe URL for 15/413/1033/initstrings.js file not found."
date: 2016-07-28
---
A strange error occurred to me today when I wanted to add a simple default SharePoint 2010 Approval workflow to a library.

I'm adding the workflow as usual:
<img src="/images/wf-error/2016-07-28_14-56-25.png" class="img-responsive" alt="adding wf">

Next, when I try to update the WF with my username from the people picker, it just clears the field after spending 1 second displaying the "uploading information to server" message and then nothing appears to happen.

Looking through the browser's network traffic to figure out what the hell is going on, I see an aspx page that for some reason does not load or display in the UI, but contains some more information:

<img src="/images/wf-error/2016-07-28_15-03-20.png" class="img-responsive" alt="error">

In plain text:
*Cannot make a cache safe URL for "15/413/1033/initstrings.js", file not found.*

With a correlation ID also below (which is handy, because I am sending this straight to Microsoft).

The JS file provided in the error does not even exist (the 413 directory), it's present under 15/1033/initstrings.js.

I will be reporting this error to Microsoft though, and will update this post if and when we're able to figure this out.