---
layout: post
title: "Changing the default ViaWorks logo"
date: 2016-07-25
---

# How to change the logo for ViaWorks search user interface
The search center is delivered with a set of logos that can be changed, here's how it looks out of the box:
<img src="/images/viaworks-logo/1-default-logo.png" class="img-responsive" alt="Default logo">

### Prepare your logo files
Using your logo image file, create two files in your favourite image editor (for this example I've used [GIMP](https://www.gimp.org/), which is a powerful and free image editor that suits our needs well).

If there's some whitespace in your logo, be sure to add an alpha channel to the image, then select and remove all the background color, then create two versions of the files in these two resolutions:

1. height:32px, width:32px
2. height: as you wish, width: 220x

<img src="/images/viaworks-logo/2-gimp-edit.png" class="img-responsive" alt="Edit the logo files">

### Upload the files
Upload the image files you edited/scaled to the following location:
E:\Program Files\VirtualWorks\ViaWorks\RestService\Resources\Icons\Templates
(change E:\ to whatever installation directory you used when installing ViaWorks)

<img src="/images/viaworks-logo/3-file-location.png" class="img-responsive" alt="Upload the files">

### Change the standard.html template
There are a few lines that need to be changed in the following html file. I recommend just commenting out the default settings so you can easily revert the process.
E:\Program Files\VirtualWorks\ViaWorks\RestService\Resources\Application\WebClient\Templates\Standard.html

Copy the line containing this node:

`<img ng-src="{{'resource/brand/image'|viaworksRestApiUrl}}"  height="32" width="32" />`

and modify it like this:

`<img ng-src="{{'resource/template/icon/32x32_logo_filename.png'|viaworksRestApiUrl}}"  height="32" width="32" />`

Copy the line containing this node:

`<img ng-src="{{'resource/brand/image/product'|viaworksRestApiUrl}}"/>`

and modify it like this:  
`<img ng-src="{{'resource/template/icon/220x220_logo_filename.png'|viaworksRestApiUrl}}"/>`

<img src="/images/viaworks-logo/4-edits.png" class="img-responsive" alt="Edit the template file">

Click save to submit your changes (might fail if you are not running your editor as Administrator)

### Verify that your changes are successfully deployed

<img src="/images/viaworks-logo/5-success.png" class="img-responsive" alt="Verify that your change is successful">