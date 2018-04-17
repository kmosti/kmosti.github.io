---
layout: post
title: "SharePoint Online webhook handler"
date: 2018-04-16
---

This post goes more into detail on how to pragmatically handle SharePoint Online webhooks - and how to work with them.

There are several great posts already detailing how you can create a webhook, how you can receive the event and how to respond - so my post will not go into details on these issues, but I will focus more on tying these things together in an Azure based solution:

1. Setting up the webhook handler using an azure site and Node.js
2. How to use the changetoken returned from the lists
3. How to store this changetoken in an Azure MySql database so that we can query this token next time an event is triggered
4. How to handle the fact that SharePoint webhook subscriptions actually expire - for this we will use the same site as above, through a WebJob that runs on a daily schedule

## First though: what are webhoks?

SharePoint webhooks are simply put HTTP POST requests issued by Office 365 when something happens in a SharePoint list or library. They are an information object that tells a web service:

>"Hey! Something happened on this subscription in this site!"

They are quite robust, in the sense that they will re-try this message up to five times in the event that it does not work the first time around (of course, we will try to avoid this ever happening and luckily Azure services can be easily run in High Availability mode so this really helps).

If you need more details on what they are though, I can recommend this presentation by Elio Struyf, where he goes into a bit more details: https://www.slideshare.net/DIWUG/spsnl17-getting-notified-by-sharepoint-with-the-webhook-functionality-elio-struyf (or just have a look at the MS [documentation](https://docs.microsoft.com/en-us/sharepoint/dev/apis/webhooks/overview-sharepoint-webhooks)).

## So how can I set up a site in Azure that will receive these messages?

You can of course use whatever language you want, as long as your webhook endpoint can reply to a POST validation query within 5 seconds, but in my example I use Node.js.

First, we will need an Azure App Service - from the Azure portal, select the app service and set it up with your preferred pricing tier (I used B1 Basic for testing, but minimum for production would be the Standard tier so you can have some additional features like scaling out your app for HA).

We will also need a DB layer, I opted for mysql in my example (because it is a new service and I wanted to try it), but you can use Azure SQL database too which is probably cheaper in the long run. The difference in the code should not be that great if you want to go that route instead.

I chose the general purpose pricing tier with the lowest specs, for the purpose of managing webhooks this should be more than enough for medium to large environments. Note down the username/password and hostname when setting up the DB.

Once you have deployed your MySQL database, select the connection security blade and  open the Access for Azure services and optionally add your current IP to the Firewall rules to allow local testing.

Next, you will need to create the DB we will use, which can easily be done using the Cloud Shell in Azure (the "terminal" icon next to your username in the top right corner).

Connect to mysql like this:

```
mysql --host <fully qualified server name> --user <server admin login name>@<server name> -p
```

Then create your database like this:
```
CREATE DATABASE 'sharepoint_webhooks'
```

We will create the table from Node, so this is all you need to do for now.


## Deploying your application to Azure

First you need to clone my sample application here:  
https://github.com/kmosti/SP-Azure-Webhooks-Handler

````
git clone https://github.com/kmosti/SP-Azure-Webhooks-Handler
````

Then build the application by using npm -install
```
npm install
```

Next, you need to change the config.js file along with some other variables like the hostname to your SharePoint Online tenant, the code is rather simple so that should not be an issue.

Once that is done, you can start the application in debug mode, which should make the page http://localhost:1337/index available as a test.

If that page loads, you can try deploying the table (double check your mysql config and Azure FW security first!) by simply visiting http://localhost:1337/reset_mysql (check the VS Code debug output to make sure!).

If all that works as it should, we can move on to testing the SharePoint parts of the application - to test this locally, you will need to install [ngrok](https://ngrok.com/), which is a tunnelling service that makes your locally running application available through a ngrok address.

Run:
````
ngrok http 1337
````
to start the tunnel, and note down the https address.

Next open Chrome and if you do not already have the addin called "SP Editor", [install it](https://chrome.google.com/webstore/detail/sp-editor/ecblfcmjnbbgaojblcpmjoamegpbodhd) (it is a great extension and a must have for SharePoint developers).

With that installed, navigate to your SharePoint site where you wish to test your webhook, then open chrome dev tools and select the SharePoint pane:

<img src="/images/webhooks/add_sp_webhook.png" class="img-responsive" alt="Add SharePoint webhook">

Select the webhook section, then the list you want to try this on and paste in the ngrok tunnel URL and select "Add".  
You should see a validation query to your local server, and that should validate the webhook.

Your list should now send a POST query containing the webhook information whenever you make a change to something in your list (add/modify/remove).

So with that set up, make a change and monitor your server - it should now receive the webhook, then proceed to send a call to the site asking for the change that occurred and then register that webhook subscription in your DB. Verify this by navigating to http://localhost:1337/mysql (you should have one new row added).

What happened was basically that we connected to the sharepoint site, queried the site for which list contains the webhook subscription, asked that list for a new changetoken (a changetoken is how we can query the site for "what has changed since the last time we asked"), then when that is done we add this information to the DB so we can easily query the site later with the correct list, subscription and changetoken the next time around.

Each subsequent trigger will now make the changeToken value update.

Try making an additional change to your list, and verify that the changeToken value has been updated in your DB.

If all these local tests are successfull, you are ready to deploy your site to Azure. Before you do that though, you might want to disable the route which drops and recreates your table.

The deployment can be made in a variety of ways, but for simple testing, I would suggest trying the "Azure App Service" extension to VS Code. Install it, then connect it to your Azure subscription by logging in, then you should see a deployment icon in VS Code:

<img src="/images/webhooks/deploy_to_app_service.png" class="img-responsive" alt="Deploy to app service">

Follow the wizard, and select the windows option when asked.  
The application will be deployed by zipping your project folder, then uploading it to your Azure site - you will be notified in the debug console once the deployment is complete - try visiting the site <url>/index to verify that it is running.

END OF PART 1