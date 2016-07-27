---
layout: post
title: "Using datatables.net to display sites in SharePoint Online"
date: 2016-07-27
---

If you've ever managed a large-ish SharePoint environment, you will probably know that after a while it can become a big problem just finding out where site x is located, and while search usually does a good job of finding your stuff, what you really want is one place where you can see all the sites you have access to along with some information about these sites.

In this post, I will show how to leverage datatables.net and the SharePoint Client Side Object Model (CSOM) and the SharePoint search REST API to generate an end user friendly list of all sites the user has access to. We will enrich the sites with metadata too, which is a good thing if you have a lot of them.

The tools we will need to do this are:

1. [Datatables.net](https://datatables.net/)
2. [Moment.js](http://momentjs.com/)
3. [SharePoint Online Client Components SDK](https://www.microsoft.com/en-us/download/details.aspx?id=42038)
4. [SharePoint Online Management Shell](https://technet.microsoft.com/nb-no/library/fp161372.aspx)

You will also (of course!) need access to a SharePoint Online administrator account.

The steps we will take here will build a proof of concept, but I trust that you will be able to figure out how to adapt this into your own environment.

On a high level, what we will do will be the following:

1. Deploy a bunch of sites using SPO Management Shell
2. Enrich these sites with metadata to their property bag
3. Configure SP Search to map these properties
4. Query the SP Search API and present our results in datatables.net

## Step 1, install all dependencies and make sure you can add custom scripts in SharePoint Online.
Visit the above links for setting up the SharePoint PowerShell dependencies, then check to see if custom scripts are enabled (follow [this guide](https://support.office.com/en-us/article/Turn-scripting-capabilities-on-or-off-1f2c515f-5d7e-448a-9fd7-835da935584f?ui=en-US&rs=en-US&ad=US&fromAR=1) to check).

## Step 2, Deploy a bunch of sites and enrich them with metadata
For this example, I started off just creating a list of sites with some metadata in a spreadsheet that I exported to a CSV file which you will find [here](https://github.com/kmosti/datatables-spsearch/blob/master/sposites_poc.csv).

After having saved that file locally, you now need to download this PowerShell script and change the various variables within it to match your environment:

[https://github.com/kmosti/datatables-spsearch/blob/master/CSOM-PowerShell/Set-SPO-web-property-CSOM.ps1](https://github.com/kmosti/datatables-spsearch/blob/master/CSOM-PowerShell/Set-SPO-web-property-CSOM.ps1)

Next, run the script (maybe try with just 2 or 3 sites first, just set the rest to Provisioned=Yes to ignore them in your script), this will deploy your sites and set the metadata in the site's property bag.

<img src="/images/datatables-spsearch/1-deploy-sites-and-metadata.png" class="img-responsive" alt="Running your script">

## Step 3, map the properties in SharePoint search
Now this step may take a little while, because while we can do a lot of stuff with SharePoint Online, managing the crawl schedule is beyond what we can do.

To speed up the process somewhat though, we can request to have one of our site collections be reindexed, which can be done by going to:  
Site Settings -> Search -> Search and offline availability -> Reindex site

Once that is done, you might want to do something else for an hour or so (it took an hour when I prepared this post, but there's no guarantees here, so this may very well take even longer).

To check if you can proceed, we need to check if the properties are available in the SharePoint search schema as a crawled property.

Go to https://TENNANT-admin.sharepoint.com/ -> search -> Manage Search Schema -> Crawled Properties

Next do a search for one of the properties, e.g. "ProjectShortName", if it's been picked up by SP, you should see this result:
<img src="/images/datatables-spsearch/2-crawled-property.png" class="img-responsive" alt="crawled-property">

Once you're there, it is time to switch over to the Managed Properties tab and create Managed Properties for all your crawled properties, just click "New Managed Property", then set them up as I've shown here:
<img src="/images/datatables-spsearch/3-managed-properties.png" class="img-responsive" alt="managed properties">

Note that you *absolutely have to* tick off "retrieve". The other options I've selected are recommended, but not required in this example.

Once that is done, we need to trigger another Reindex of the site in order for SharePoint to populate these Managed Properties with data (you do not actually have to trigger a reindex, but it will speed up the process).

## Step 4, deploy the necessary JS files to SharePoint
Visit my [GitHub project page](https://github.com/kmosti/datatables-spsearch) and download these items:
<img src="/images/datatables-spsearch/4-resource-files.png" class="img-responsive" alt="resource-files">

Switch out the search query string and upload them to SharePoint, as shown here in my Site Assets library:
<img src="/images/datatables-spsearch/5-upload-resource-files.png" class="img-responsive" alt="upload-resource-files">


## Step 5, test and verify the search results
Now there *might* be some changes that you need to perform on the file spo.search.functions.js, since we're now going to query SharePoint Search to return our sites in datatables.net.

For this, I recommend using a Chrome App called [Advanced Rest Client](https://chrome.google.com/webstore/detail/advanced-rest-client/hgmloofddffdnphfgcellkdfbfbjeloo), which will provide us with a good way to fine tune our search query and preview the returned JSON.

You will also need to install the [ARC cookie exchange](https://chrome.google.com/webstore/detail/arc-cookie-exchange/apcedakaoficjlofohhcmkkljehnmebp) so that the app can authenticate to SharePoint Online.

Open the app, then select "Use XHR" to enable it to authenticate (you need to log into your SharePoint site in Chrome first!), and paste in your request url.
In my example (just switch out "spzealot" with your own tennant name):

**https://spzealot.sharepoint.com/_api/search/query?querytext='contentclass:STS_Site+path:https:%2f%2fspzealot.sharepoint.com%2fteams%2fproj*'&trimduplicates=false&rowlimit=10&selectproperties='Title%2CDescription%2CLastModifiedTime%2Cpath%2CAAAProjectShortName%2CAAAProjectCategory%2CAAAProjectManager%2CAAAProjectClient%2CAAAProjectNumber'**

Now we can drill down into our search results, and if you've done everything correctly thus far, you should see values in the node:
d.query.PrimaryQueryResult.RelevantResults.Table.Rows.results

You will now have to verify that the results correspond like this within the function allRecentDocuments:
<img src="/images/datatables-spsearch/7-map-keys.png" class="img-responsive" alt="map-keys">

After having tested and verified, upload your changes.

## Step 6, displaying the results on a page

This step requires that the use of custom code is enabled on your tennant or your site (ref Step 1), if you just made this change, then you might need to wait a while since the change is implemented once per day, so in that case just try again tomorrow.

Go to the page where you want to display the sites, then upload a Content Editor Web Part:
<img src="/images/datatables-spsearch/8-insert-cewp.png" class="img-responsive" alt="insert-cewp">

Next, select the web part, then open the web part tool pane and select "Web Part Properties":
<img src="/images/datatables-spsearch/9-open-cewp-props.png" class="img-responsive" alt="open-cewp-props">

In the "Content Link" field, paste in the url to the list_webs.htm file:
<img src="/images/datatables-spsearch/10-link-cewp-to-file.png" class="img-responsive" alt="link-cewp-to-file">

Click OK, then save/publish your page.

If you have now done everything correctly, you should now see this as a result:

<img src="/images/datatables-spsearch/11-success.png" class="img-responsive" alt="success">

And that's it really!

Some of the cool things about this approach is that you can actually display thousands of sites this way and it will be fast thanks to the very fast response from the search REST API, the code will also loop through the result set so that there really is not a set limit on how many sites you can display in one table.
Since search results are also security trimmed, you will always be able to open the links in the table, and sites you do not have access to will not be displayed.

With datatables, you get a bunch of functionality built right into the solution - the filtering (search) is especially nice, and I've also added date handling (using moment.js) and sorting in the JS file so that you can select to sort by it and have the most recently changed sites on top.

You should also definitely check out the [datatables.net](https://datatables.net/) homepage, blog and forums for more stuff you can do with datatables, like [multi-column ordering](https://datatables.net/release-datatables/examples/basic_init/multi_col_sort.html) , [language options](https://datatables.net/examples/basic_init/language.html) or [other cool stuff](https://datatables.net/examples/) - if you do get some cool ideas, please drop me a line in the comments below and tell me more!

I hope that this post has been somewhat informative and appreciate your feedback.