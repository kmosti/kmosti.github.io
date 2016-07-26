---
layout: post
title: "JS Link breadcrumb path"
date: 2016-07-26
---

This post details how to add a breadcrumb path in flat library views, which is often an issue especially in large libraries with a complicated folder structure.

SharePoint views by default will display folders in regular folder structure, like this:

<img src="/images/JS-link-folderpath/1-default-view.png" class="img-responsive" alt="Default view">

There are times though, like e.g. when you wish to use the column sorting/filtering or grouping by columns where it is much better to ignore the folders. Fortunately, SharePoint has this functionality built in and the option can be set per view:

<img src="/images/JS-link-folderpath/2-edit-view.png" class="img-responsive" alt="Edit view">

The downside to this though, is that there really is not an easy way (short of inspecting the url of the file) to see *where* the file is located, and that is where using client side rendering comes in handy.

To do this, we will add a blank column to a flat view, then upload a JavaScript file to SharePoint and set the view to use this file to render the column with a nice little breadcrumb path that will tell the user in which folder the file is located.

## Step 1

Create your view, be sure to set "Show all items without folders" and add a column to your library called simply "Folder" (this column should be added only to this view).

## Step 2

Copy this code into a file and save it as "Parse_Folder.js" to a site assets library or similar (in this example I have uploaded it to the Site Assets library on the top level site in the site collection):

	//Script to parse folder level into a placeholder column.
	//Required: Column named "Folder"

	(function () {
	    var pathContext = {};
	    pathContext.Templates = {};
	    pathContext.Templates.Fields = {
	        "Folder": { "View": getFolderPath }
	    };

	    SPClientTemplates.TemplateManager.RegisterTemplateOverrides(pathContext);

	})();

	function getFolderPath(ctx) {
	    var folderPath = "";    
	    var libraryPath = ctx.listUrlDir; //get the relative path of the library
	    var itemPath = ctx.CurrentItem.FileRef; //get the item's relative path
	    var folderArray = itemPath.split(libraryPath)[1].split("/"); //split item's path into array starting from root folder
	    var folderArraySlice = folderArray.slice(0, folderArray.length-1); //slice array to only folders, ignore sites and filename
	    var len = folderArraySlice.length;
	    for (var i=0;i<folderArraySlice.length;i++)
	        {
	            if (i == len-1) { //the last folder in the array
	                folderPath += folderArraySlice[i];
	        }
	        else {
	            folderPath += folderArraySlice[i];
	            folderPath += "/"; //add separator when there are more folders
	        }
	    }
	    return folderPath
	}

## Step 3

Open your view, then select "Edit page" and select "Edit Web Part" in the web part's context menu:

<img src="/images/JS-link-folderpath/3-edit-webpart.png" class="img-responsive" alt="Web part context menu">

Next, in the tool pane under "Miscellaneous", paste the link to the JS file, which should start with a SharePoint variable ~sitecollection, like this:

~sitecollections/siteassets/Parse_Folder.js

<img src="/images/JS-link-folderpath/4-edit-webpart.png.png" class="img-responsive" alt="Tool Pane configuration">

Click OK and save/publish your page

## Result

You might need a refresh of the page, but it should now look pretty much like this:

<img src="/images/JS-link-folderpath/5-result.png" class="img-responsive" alt="Result">

I hope this is helpful to someone, and if you have any questions just leave me a comment below.