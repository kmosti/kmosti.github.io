---
layout: post
title: "SharePoint Online webhook handler part 2"
date: 2018-04-17
---

This is the second part of an article that details how to manage SharePoint webhook subscriptions through Azure Apps. (you can read the first part [here](http://spzealot.com/2018/04/16/SharePoint-Online-webhook-eventhandler-part-1.html))

In the first part, we deployed the Azure site that handles the webhook subscriptions and stores them in a MySQL database, in this second part we will look into how to configure an automatic renewal of subscriptions - thus making sure they never expire.

The important part about the expiration date is that you can configure this date to be at a maximum of 6 months from today's date, which means that an automated job needs to pick them up as they are closing in on this date and then update them with some new date which is less than 182 days.

## Azure WebJobs

Each Azure app site has a separate section where you can configure WebJobs to run. WebJobs are pretty much identical to Azure functions, including the trigger options.

Since we already have a DB that stores subscriptions, including their site url, list id and expiration date, we can now configure a daily webjob that queries the database for subscriptions about to expire, then update the expiration dates of those using the sp-pnp-js.

The solution I made for this purpose can be found here: https://github.com/kmosti/SP-Azure-Webhooks-renewer

To install:

```
git clone https://github.com/kmosti/SP-Azure-Webhooks-renewer
npm install
```

Again, change the config file with your own values.

Once that has been done, you can now test then deploy your webjob. In this instance though, we cannot use the VS Code extension (it does not integrate with WebJobs), however we can do it by zipping the entire folder and uploading it (include the node_modules directory in the zip file). So for mac users just use Terminal, navigate to the root folder then run

```
zip -r ../webjob.zip *
```

For windows users, just right click and zip the folder.

In the Azure portal, navigate to the App Service, find the WebJobs blade and click "New"

<img src="/images/webhooks/webjob_1.png" class="img-responsive" alt="Add WebJob">

Enter a name, select type "Triggered", upload the zip file and enter a CRON expression that will trigger the WebJob on a daily schedule. In my test I run it every night at 0100, in which case my CRON expressions is:

```
0 0 1 * * *
```

Wait for the deploy to finish, then test it if you want by manually modifying a row in the database to a date less than your update time window and run the WebJob locally.

NOTE: in order for WebJobs to run on a daily schedule, your App service needs to be set to "Always On":

<img src="/images/webhooks/webjob_2.png" class="img-responsive" alt="Always On">

To monitor your web job, you can open select your WebJob and click the "Logs" pane, you should see quite detailed logs and get the status.

## Registering webhooks

I did not touch on this point except to say you can register a webhook from the SP Editor Chrome extension, but you should know that a webhook can be registered either by a REST query, using sp-pnp-js or if you already are using the PnP Provisioning engine, you can easily add the webhook registration in the PnP Scheme for any list or library by adding the following XML:

```
<pnp:Webhooks>
    <pnp:Webhook ServerNotificationUrl="https://<yourwebhookendpoint" ExpiresInDays="120" />
</pnp:Webhooks
```

Or by using [PnP PowerShell](https://docs.microsoft.com/en-us/powershell/module/sharepoint-pnp/add-pnpwebhooksubscription?view=sharepoint-ps):

```
Add-PnPWebhookSubscription -List MyList -NotificationUrl https://my-func.azurewebsites.net/webhook
```

And probably a few other ways too.

## What to do next

This solution does not actually do much, it serves more as a base or hub for webhooks. In order to actually do things, you would first need to add some code to get the change from the list then do some action based on that. That action may be simply triggering an Azure function with parameters obtained from the item, it may be triggering a Flow or Logic App or running some custom code to do other stuff. I will leave that up to the reader, but feel free to drop me an email or post a comment with ideas you would like me to explore in another blog post.