---
layout: post
title: "Using PnP JS to update security for all items in a list"
date: 2018-05-02
---

I had this issue after a ShareGate migration where I mistakenly opted to preserve the permissions when migrating content from a fileserver - this caused most of the files to get unique permissions and a lot of the users could not see the migrated content as we planned.

Since this was a rather large library, carefully planned with the use of indexed views to handle the list item view threshold, I was not too keen on migrating the content again, especially since the customer had a somewhat limited network so that I would have to do this during my weekend.

I did not either want to run PowerShell because it is too slow for handling a list with 65 000 items. My solution was to use the SharePoint Chrome add-in's JS console, which basically runs 6 concurrent threads so that I can really turn up the tempo.

Here is the script to be used (replace the listTitle and configure the filter):

```javascript
import pnp from "pnp";

const listTitle = "List title";

async function breakPermission (itemID, currentCount, totalLength) {
	let item = await pnp.sp.web.lists.getByTitle(listTitle).items.getById(itemID);
	item.resetRoleInheritance().then( () => {
		if (currentCount == totalLength) {
			console.log("Done!");
		}
	}).catch( err => {
		console.dir(err);
		console.log("retrying permission update for " + itemID);
		item.resetRoleInheritance().then( () => {
			console.log("retry for " + itemID + " successfull!");
			if (currentCount == totalLength) {
				console.log("Done!");
			}
		}).catch( err2 => {
			console.dir(err2);
			console.log("retried permission update for " + itemID + " -- SECOND ERROR");
		});
	});
}

pnp.sp.web.lists.getByTitle(listTitle)
.items
.filter("Created ge datetime'2010-01-01T00%3a00%3a00' and Created le datetime'2011-01-01T00%3a00%3a00'")
.top(5000).get().then( res => {
	let counter = 1;
	const length = res.length;
	console.log(length);
	for (let r of res) {
		counter ++;
		breakPermission(r.Id, counter, length);
	}
});
```

And if everything works, this should be the output:

<img src="/images/permissions/output.png" class="img-responsive" alt="console output">