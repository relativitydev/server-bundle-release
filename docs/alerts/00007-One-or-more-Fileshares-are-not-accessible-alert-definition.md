# One or more Fileshares are not accessible

## Description: 
Alert is true if the one or more resource server network share is inaccessible.

## Alert Details:
**Alert ID:** 8e754e39-c73f-480f-a26e-a9895c624426

**Tags:**
Each tag should follow "key:value" format.

|Tag Name|Tag Key|Tag Value|
|--|--|--|
|Alert Type|Type|Infrastructure|
|Alert Group|Group|Network Connectivity|
|Alert Navigation Type|PageType|Dashboard|
|Alert Navigation param|PageID|5564a18f-f27e-4dfb-a296-d41e83f53ae8|
|Created By|CreatedBy|Relativity|
|Resolution Text|ResolutionText||
|Resolution URL|ResolutionURL|/docs/alerts//docs/alerts/00007-One-or-more-Fileshares-are-not-accessible-alert-resolution-sop.md|

## Metric Details:
**Metric Name:** relsvr.resourceserver_networkshare.accessible

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.mount_point|File share location||
|labels.state||accessible/inaccessible|
|labels.system_filesystem_display_name|File share resource server display name||

## Rule details
**Alert Condition Description:** Alert is true if the one or more resource server network share is inaccessible for at least 90 seconds.

|Name|Value|Description|
|-|-|-|
|Rule Type| Elastic Query||
|Data View| metrics-*||
|Filter Query|relsvr.resourceserver_networkshare.accessible: 0|Network share is inaccessible|
|Group| Count|number of Network share locations are inaccessible|
|Threshold| > 0| Count greter than 0, alert triggers|
|Time Window| 90 sec| Verified data for last 90 sec|
|Frequency| 30 sec |Checks for each 30 seconds|

## Visualization link
Kibana dashboard link

