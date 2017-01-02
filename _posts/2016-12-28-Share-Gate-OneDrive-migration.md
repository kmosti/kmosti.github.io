---
layout: post
title: "Share-Gate OneDrive migration"
date: 2017-02-01
---

Today's post is all about migration - tenant to tenant migration of OneDrive sites from a SharePoint perspective.

The case in question was that of a tenant which needed to switch tenant names, something you'd expect Microsoft to make a bit easier, but as it stands you basically need to create a new tenant, then move all your stuff over.

With more than 8000 provisioned sites, this proved to be a bit more work than expected, but for what it's worth I hope someone's job will be just a little bit easier after having read this post.

We decided upon using Share-Gate as the tool of choice to do the migration itself. Not only does it have a good log of the process, but it is quite robust and supports the kind of volume we need, while also not having to worry about temp storage since everything runs through Azure.

The process as detailed here presupposes that you've already synced in all your user accounts from AD to your new tenant.

## Preprovision all OD sites

To be able to do this quickly, we will need to first preprovision all OneDrive sites for our users, to do that we need to extract a CSV file with all names, emails etc.. of the users, then we need to clean up the emails, export them to text files with a max count of 200 rows, then import them into a provisioning queue. Easy peasy, right?

First things first: export all emails of your users:

<script src="https://gist.github.com/kmosti/0d3ccf9f259eaa84706299902c113262.js"></script>

This will create a neat csv file with the information we need, however you will notice that the email addresses field is just a long string of all your smtp, sip and other fields, and what we need is a neat little file with just our email addresses, so we need to clean all this up a bit and divide them.

There are probably a lot neater ways of solving this (and I welcome suggestions in the comments if you have better ways), but it took me 10 minutes to write this out so here it goes:

<script src="https://gist.github.com/kmosti/51a7be7f82761aab323d1137ddaec060.js"></script>

We now have a directory with a bunch of 200 row text files containing email addresses of our users.

Next we need to run a script to preprovision these, and this script has a max input of 200 rows (hence the trouble we went through in the last step).

Save it as provision_OneDrive.ps1 and make sure you have your PS execution policy set to a level that lets you run it.

<script src="https://gist.github.com/kmosti/79fa5d525ba48303d3b86882d69d8018.js"></script>

With 8000+ sites to provision, I noticed that it took about 2 hours to run this script, and about 24 hours before all sites were provisioned in SharePoint Online, however your mileage may vary so just make sure you confirm that they have all been provisioned before you start your migration process.


## Do an inventory, and build you input file(s)

Next, we need to map out or migration process, and for that ShareGate has an inventory tool which combined with some Excel magic will provide us with a CSV file to be used as input for the ShareGate PowerShell module.

So, run an inventory of OD sites in ShareGate for both the new and the old tenant:

<img src="/images/OD-migration/inventory.png" class="img-responsive" alt="running inventory">

Next, save these to spreadsheets, then combine them into one spreadsheet.

Create a named range of all your new sites in the new tenant, select all values in the display name and URL column, then click in the top right corner to the left of the column names and give it a name like "oldsites" or similar.

Then go to your new sites sheet and remove all columns except Display Name and URL.

Add a new row to this sheet, call it destination url or something similar, then paste the following formula into the first row:

=IF(ISERROR(VLOOKUP(A2;oldsites;1;FALSE));"null";VLOOKUP(A2;oldsites;2;FALSE))

Copy that down to your last row, and if you get any null values catch them manually and remove them. This will happen if e.g. the user account doesn't exist or similar (no need to migrate more than necessary after all).

To explain what it does, it looks for the exact value in A1 (the user's name) in your named range, then returns either null if it doesn't find it, or the value in the url field adjacent to the name if it finds the name. Excel is really neat sometimes!

So now you should have a big list with Name, Destination site and Source site - which is basically all we need to use Share-Gate's Copy-Site cmdlet. Export the list to a CSV.

Now, because this process is time consuming, what I will do is to split these CSV files into smaller portions, then run each one in a different PS thread with a separate service admin account to avoid Office 365 throttling. Basically run the processes in parallell, and in so doing reducing the time to migrate with an entire order of magnitude by simply running ten instances. To do this, I simply split the csv file into smaller files and ran them in separate PS windows.

To further speed things up, I also decided to run everything from an Azure VM in the same datacenter as my tenants, but that is not really necessary unless you do not have a client with a stable network connection ready at hand.

Resource wise, I found that a Windows 2016 server with 4 CPUs ran at about 10-20% CPU and 3-4GB memory max with six processes running (I had two processes running per service account).

## The ShareGate magic

Here is the script I have been using for the migration itself:

<script src="https://gist.github.com/kmosti/73230e26dd1c6feba038bf7dbffec7c5.js"></script>

Save that script, then to run it, just start a PS session and run it like this example (or run get-help .\CopyOneDriveSites.ps1 -full for more info):

.\CopyOneDriveSites.ps1 -DestinationSPOAdminUrl https://newcontoso-admin.sharepoint.com -SourceSPOAdminUrl https://oldcontoso-admin.sharepoint.com -InputfilePath C:\input.csv -newadminuser admin@newcontoso.onmicrosoft.com -oldadminuser admin@oldcontoso.onmicrosoft.com -neWAdminPwFile C:\newadminpw.txt -oldAdminPwFile C:\oldadminpw.txt

Start as many sessions per service account as you need, then sit back and relax for a few days while ShareGate moves the stuff you need.

Here is a screenshot that shows it in action:

