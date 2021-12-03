---
title: "AzureSQL Failover Group"
date: 2019-04-26T14:36:18-05:00
draft: false
toc: false
images:
tags:
  - azure
  - failover-group
  - sql
---

Failover Groups (FG) are one of the features of Azure SQL that sprinkle magic dust upon it -- set one up, add a database to it, and you're automatically replicating to a remote region of your choice, and can failover at will if you like. An FG requires a Primary and a Secondard server: The primary is easy -- it's the one the databas(es) you want to protect are hosted on; the secondard is any other database in any other region (must be another region), where you want the FG to asynchronously replicate your database to, and where you'd like to automatically failover to if the primary region were to become unavailable.

It's pretty easy to go about creating an FG, and these steps below walk you through it:
 
#### Create failover group
`az sql failover-group create --name myfailovergroup --server myprimarydbserver --partner-server mysecondarydbserver --resource-group myRG`

#### Add existing DB to the failover group
`az sql failover-group update -n myfailovergroup --resource-group myRG --server myprimarydbserver --add-db db_test`
 
And that's it. Modify the script to suit the specific names of resources you have, and you're good to go. The FG jumps to life immediately and starts asynchronously replicating data to the secondary region, and you'll be able to even simulate a failover once that's done. Remember, the first time will likely take you the longest to replicate, as it's basically moving all the data from primary to secondary.