---
title: "VM Uptime Log Analytics"
date: 2019-10-25T14:40:21-05:00
draft: false
toc: false
images:
tags:
  - log-analytics
  - vm-uptime
---
Log Analytics is an excellent data aggregation tool for service telemetry, and keeping some quick queries easily accessible is useful.

Here's how you'd get VM Uptime via Log Analytics:

 
````
let start_time=startofday(datetime("2019-10-01 00:00:00 AM"));
let end_time=endofday(datetime("2019-10-31 11:59:59 PM"));
Heartbeat
| where TimeGenerated > start_time and TimeGenerated < end_time
| summarize heartbeat_per_hour=count() by bin_at(TimeGenerated, 1h, start_time), Computer
| extend available_per_hour=iff(heartbeat_per_hour>0, true, false)
| summarize total_available_hours=countif(available_per_hour==true) by Computer
| extend total_number_of_buckets=round((end_time-start_time)/1h)
| extend availability_rate=total_available_hours*100/total_number_of_buckets
````

