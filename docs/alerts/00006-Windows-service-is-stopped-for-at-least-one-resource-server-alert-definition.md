# Windows Service is stopped for at least one Resource Server

## Description: 
Alert is true if the Windows Service is stopped for at least one agent server, web background processing server, worker manager server or analytics server.

## Alert Details:
**Alert ID:** d14f3e28-fa2b-4df8-8b02-4d35d45bb800

**Tags:**
Each tag should follow "key:value" format.

|Tag Name|Tag Key|Tag Value|
|--|--|--|
|Alert Type|Type|Infrastructure|
|Alert Group|Group|Server Health|
|Alert Navigation Type|PageType|Dashboard|
|Alert Navigation param|PageID|cd200ee0-1e61-4645-8220-83ce82914a71|
|Created By|CreatedBy|Relativity|
|Resolution Text|ResolutionText||
|Resolution URL|ResolutionURL|/docs/alerts/00006-Windows-service-is-stopped-for-at-least-one-resource-server-alert-resolution-sop.md|

## Metric Details:
**Metric Name:** relsvr.windows_service.running

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.name|Name of Service||
|labels.startup_mode|||
|labels.state||accessible/inaccessible|

## Rule details
**Alert Condition Description:** Alert triggers on at least one windows service stopped for at least 90 seconds.

|Name|Value|Description|
|-|-|-|
|Rule Type| Elastic Query||
|Data View| metrics-*||
|Filter Query|relsvr.windows_service.running : 0|Windows service is stopped|
|Group| Count|number of Windows service is stopped|
|Threshold| > 0| Count greter than 0, alert triggers|
|Time Window| 90 sec| Verified data for last 90 sec|
|Frequency| 30 sec |Checks for each 30 seconds|

## Visualization link
Kibana dashboard link

