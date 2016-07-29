---
layout: post
title: "BindTuning basic bootstrap theme in SharePoint Online"
date: 2016-07-29
---

Today I figured i would mention a topic that comes up a lot with SharePoint, especially with designers that are just starting out and just need something basic to build upon to create a nice looking SharePoint site: basic bootstrap design.

Now if all you wish is to make SharePoint more *responsive*, then there might be a simpler option at hand from the hands of the PnP team:
[https://github.com/OfficeDev/PnP-Partner-Pack/tree/master/OfficeDevPnP.PartnerPack.SiteProvisioning/OfficeDevPnP.PartnerPack.SiteProvisioning/Templates/Responsive](https://github.com/OfficeDev/PnP-Partner-Pack/tree/master/OfficeDevPnP.PartnerPack.SiteProvisioning/OfficeDevPnP.PartnerPack.SiteProvisioning/Templates/Responsive)

However if you want the simplest, cheapest way to introduce true bootstrap design on your SharePoint site, then check out [BindTuning](http://bindtuning.com)'s free Bootstrap theme:

[http://bindtuning.com/cms/sharepoint/office-365-v2013/theme/TheBootstrapTheme/page/Default/customize](http://bindtuning.com/cms/sharepoint/office-365-v2013/theme/TheBootstrapTheme/page/Default/customize)

BindTuning is a company of designers out of Lisbon that build design packages for a lot of platforms, including SharePoint, and their model is simple:

1. Select a base theme (the bootstrap theme is free, but there are many other reasonably priced themes available)
2. Use a browser based configurator to customize it
3. Run it through their design engine and out on the other hand comes a sanboxed solution that you can deploy on your SharePoint site collection

An important note here is that this *will* introduce a custom masterpage to your site, which *may* not be a great option for collaboration sites (team sites and similar, where the above mentioned responsive design may be better suited), however for publishing portals (intranets and similar) it might actually be a good thing because it gives you another layer of control before Microsoft updates hits your users and introduces design changes.

Ok, so on to my little guide to show you how to deploy the theme in SharePoint Online:

## Step 1, open the configurator and make your design customizations
Right away you will see a live preview with a side panel for customizations.

In the first tab, select a base color scheme (with current trends, less is more as they say, so try selecting one with one or just a few base colors)

<img src="/images/bootstrap/1-configuration.png" class="img-responsive" alt="Configuration tool">

Now if you already have a public web site with a company color scheme that you wish to use, BindTuning does have a "Magic wand" tool that allows you to fetch the design and apply it to your preview, try it if you want with any web site you wish, here I have selected the color scheme from [http://www.ferrari.com/](http://www.ferrari.com/):

<img src="/images/bootstrap/2-magic.png" class="img-responsive" alt="Magic tool">

Next just drill down into the "customize" tab to set colors/width/sections/text/tokens etc.. If you are familiar with design these things should be quite familiar, and if not then there's a live preview to show you what changing settings does. Easy right?

Once you are done, try resizing your page width or emulating different devices using the preset bottom buttons for desktop/tablet/mobile and if everything looks good, **save your configuration** (it'll get saved to your bindtuning account), then hit the Get theme button on the bottom:

<img src="/images/bootstrap/3-getit.png" class="img-responsive" alt="Get your theme">

Your theme will now enter a queue to be processed, so depending on the current load it should be ready within ten to fifteen minutes (they will also send you an email notification once it's ready):

<img src="/images/bootstrap/4-getit.png" class="img-responsive" alt="Get your theme">

Done!

<img src="/images/bootstrap/5-getit.png" class="img-responsive" alt="Get your theme">

## Step 2, import your theme to SharePoint
Before you upload it, there are a few things that you need to take care of on the site collection where you upload your theme:

1. Turn off these features (Site settings -> Site features):
	1. Minimal download strategy
	2. Mobile Browser View
2. Turn on the site collection feature **SharePoint Server Publishing Infrastructure**
3. Turn on the site feature **SharePoint Server Publishing**

So now we're ready to deploy our solution and activate our bootstrap package, but first we might want to just have a look inside our solution to see what we will deploy, and for this we can use any archive program that supports cab files.

The package you download will be a .zip file with a wsp package, but a wsp file is basically just a compressed container that contains all the files we will deploy (plus some manifest files that tells SharePoint how to deploy these things, but that is not important).

I made a copy of the file, renamed it to .cab and unpacked it. The results are shown here:

<img src="/images/bootstrap/6-wsp-contents.png" class="img-responsive" alt="wsp-contents">

As you can see, it will deploy a bunch of page layouts and 3 masterpages in addition to css/image and js files that will be deployed to the Style Library. Once the package is deployed, you will be able to open and edit these files as you wish (there are also uncompressed css files for your theme that you can modify and edit, unless you want to just make your own, that is).

You can also make changes right here, then compress the files back to cab and rename the file to wsp if you want a quick and dirty way of editing a wsp package.

Step 3, deploy your theme

1. Navigate to Site settings -> Solutions
2. Select upload solution, then upload your wsp file
3. Once the file is uploaded, a dialog with an "Activate" button is presented - click the button
4. This might take a little while (it will deploy all the files), so just wait for a few minutes

Once your solution is deployed and activated you should see something like this:

<img src="/images/bootstrap/7-deploy-wsp.png" class="img-responsive" alt="deploy-wsp">

Now to use the new solution, you need to change out the default masterpage with the bootstrap page.

Go to Site Settings -> Master page, then select your new masterpage in the drop down menu.

<img src="/images/bootstrap/8-change-masterpage.png" class="img-responsive" alt="change-masterpage">

You should now see that the layout changes, and you can start using your theme - to try out some of the page layouts, just go to the Page library and create a new Article page, then select one of the page layouts (I've chosen (Article Page)Image on left below).

<img src="/images/bootstrap/9-deployed.png" class="img-responsive" alt="example">

This is just scratching the surface a bit though, as you can now fully use all the bootstrap functionality within SharePoint - including the grid, the classes, the modals and all other features.

If you go back to the [live preview page on BindTuning](http://bindtuning.com/cms/sharepoint/office-365-v2013/theme/TheBootstrapTheme/page/Default/customize), you will also notice that the preview functions as an interactive documentation on how to use bootstrap features, if you want to e.g. add a slider to your front page, just go to Widgets -> Sliders and select "view code" on the example you want to use.

This concludes this mini-tutorial, but if you have any questions you can ask them in the comments below or just contact BindTuning themselves.