---
layout: post
title: "Tuning ViaWorks resource consumption"
date: 2016-08-02
---

ViaWorks is a resource hungry monster, it will chew up any CPU you throw at it and use it to crawl through your data repositories as fast as possible - generally this is a good thing during a full crawl where you want to add your data to the search index as fast as possible, however once you are done and you are in "daily operation", then you might want to lessen ViaWork's resource appetite.

Fortunately, this is rather easy to do by just tweaking a few DB values in the config.via_works_server_setting table.

Before you start, monitor the CPU usage in the task monitor as well as how many Postgres threads are currently in use (these should be reduced afterwards).

Open up PgAdmin3 (in ProgramFiles\VirtualWorks\ViaWorks\PostgreSQL\bin), then add these lines to the table **config.via_works_server_setting**:

<table class="table">
	<thead>
		<tr>
			<th>via_works_server_setting_id</th>
			<th>server_id</th>
			<th>setting_name</th>
			<th>setting_value</th>
			<th>data_type</th>
			<th>description</th>
			<th>encrypted</th>
			<th>creation_date_utc</th>
			<th>last_modified_date</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>1</td>
			<td>1</td>
			<td>FetchIdlePollDelayMilliseconds</td>
			<td>60000</td>
			<td>int</td>
			<td>FetchIdlePollDelayMilliseconds</td>
			<td>FALSE</td>
			<td>2016-01-01</td>
			<td>2016-01-01</td>
		</tr>
		<tr>
			<td>2</td>
			<td>1</td>
			<td>MaxCheckActiveFetchRequestsDelay</td>
			<td>10000</td>
			<td>int</td>
			<td>MaxCheckActiveFetchRequestsDelay</td>
			<td>FALSE</td>
			<td>2016-01-01</td>
			<td>2016-01-01</td>
		</tr>
		<tr>
			<td>3</td>
			<td>1</td>
			<td>MinCheckActiveFetchRequestsDelay</td>
			<td>9000</td>
			<td>int</td>
			<td>MinCheckActiveFetchRequestsDelay</td>
			<td>FALSE</td>
			<td>2016-01-01</td>
			<td>2016-01-01</td>
		</tr>
	</tbody>
</table>

Make sure that the IDs of the rows are unique, next add the server_id (default is 1, if you're using just one server).  
If you want to check what the server ID is, then just check the table config.via_works_server.  
Note that the date stamps should of course be changed to whatever the current date is.

If you have more than one ViaWorks server, just repeat the process using the server_id for the other server.

Your settings should now look like this (assuming only one server):

<img src="/images/viaworks-search-tuning/1-via_works_server_settings.png" class="img-responsive" alt="Settings screenshot">

Finally, in order to update viaworks with your new settings, restart all ViaWorks fetch services then return to your task manager window. You should now see a significant reduction in CPU usage - when testing I have seen the CPU usage reduced from 80/90% on average to 10/20% on average.